# Dim Date query

let <br>
    // Noem je datum tabel: Dim_Date<br>
    // Variabelen<br>
    Startdatum = #date(2010, 1, 1),<br>
    Duur = Duration.Days(Date.From(Date.EndOfYear(DateTime.LocalNow()))-Startdatum)+1,<br>
    //Datum tabel maken<br>
    #"Lijst maken" = List.Dates(Startdatum,Duur,#duration(1,0,0,0)),<br>
    #"Tabel maken" = Table.FromList(#"Lijst maken", Splitter.SplitByNothing(), null, null, ExtraValues.Error),<br>
    #"Naam kolom aanpassen" = Table.RenameColumns(#"Tabel maken",{{"Column1", "Date"}}),<br>
    #"Set type as date" = Table.TransformColumnTypes(#"Naam kolom aanpassen",{{"Date", type date}}),<br>
    #"Jaar ingevoegd" = Table.AddColumn(#"Set type as date", "Jaar", each Date.Year([Date]), Int64.Type),<br>
    #"Maand ingevoegd" = Table.AddColumn(#"Jaar ingevoegd", "MaandNummer", each Date.Month([Date]), Int64.Type),<br>
    #"Week van jaar ingevoegd" = Table.AddColumn(#"Maand ingevoegd", "WeekNummer", each Date.WeekOfYear([Date]), Int64.Type),<br>
    #"Naam van maand ingevoegd" = Table.AddColumn(#"Week van jaar ingevoegd", "Maand", each Date.MonthName([Date]), type text),<br>
    #"Dag van week ingevoegd" = Table.AddColumn(#"Naam van maand ingevoegd", "Dag van week", each Date.DayOfWeek([Date]), Int64.Type),<br>
    #"Ingevoegde naam van de dag" = Table.AddColumn(#"Dag van week ingevoegd", "Naam van de dag", each Date.DayOfWeekName([Date]), type text),<br>
    #"Kwartaal ingevoegd" = Table.AddColumn(#"Ingevoegde naam van de dag", "Kwartaal", each Date.QuarterOfYear([Date]), Int64.Type),<br>
    #"Dag ingevoegd" = Table.AddColumn(#"Kwartaal ingevoegd", "Dag", each Date.Day([Date]), Int64.Type),<br>
    #"Samengevoegde kolom ingevoegd" = Table.AddColumn(#"Dag ingevoegd", "Q Kwartaal", each Text.Combine({"Q", Text.From([Kwartaal], "nl-NL")}), type text),<br>
    #"Sorted Rows" = Table.Sort(#"Samengevoegde kolom ingevoegd",{{"Date", Order.Descending}})<br>
in<br>
    #"Sorted Rows"<br>
<br>
# Counts<br>
<br>
= Distintcount([waarde])<br>
<br>
# Running total<br>

RunningTotalTrajecten = IF(SELECTEDVALUE(Dim_Date[date]) >= TODAY(),BLANK(),<br>
CALCULATE([Aantal Trajecten],<br>
    FILTER(ALLSELECTED(Dim_Date[Date]),<br>
    Dim_Date[Date] <= MAX(Dim_Date[Date])<br>
    ),<br>
    Dim_Date[Date] <= TODAY(),<br>
    Dim_Date[Jaar] = 2024<br>
))<br>
<br>

# Regressie analyse

Linear regression Trajecten = 
VAR Known =
    FILTER (
        SELECTCOLUMNS (
            ALLSELECTED ( 'Dim_Date'[Date] ), // Date tabel date column
            "Known[X]", 'Dim_Date'[Date], 
            "Known[Y]", [RunningTotalTrajecten] // Runing totals
        ),
        AND (
            NOT ( ISBLANK ( Known[X] ) ),
            NOT ( ISBLANK ( Known[Y] ) )
        )
    )
VAR SlopeIntercept =
    LINESTX(Known, Known[Y], Known[X])
VAR Slope =
    SELECTCOLUMNS(SlopeIntercept, [Slope1])
VAR Intercept = 
    SELECTCOLUMNS(SlopeIntercept, [Intercept])
RETURN
    SUMX (
        DISTINCT ( 'Dim_Date'[Date] ),
        Intercept + Slope * 'Dim_Date'[Date]
    )
