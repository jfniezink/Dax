# Dim Date query

let <br>
&nbsp;&nbsp;&nbsp;&nbsp;    // Noem je datum tabel: Dim_Date<br>
&nbsp;&nbsp;&nbsp;&nbsp;    // Variabelen<br>
&nbsp;&nbsp;&nbsp;&nbsp;    Startdatum = #date(2010, 1, 1),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    Duur = Duration.Days(Date.From(Date.EndOfYear(DateTime.LocalNow()))-Startdatum)+1,<br>
&nbsp;&nbsp;&nbsp;&nbsp;    //Datum tabel maken<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Lijst maken" = List.Dates(Startdatum,Duur,#duration(1,0,0,0)),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Tabel maken" = Table.FromList(#"Lijst maken", Splitter.SplitByNothing(), null, null, ExtraValues.Error),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Naam kolom aanpassen" = Table.RenameColumns(#"Tabel maken",{{"Column1", "Date"}}),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Set type as date" = Table.TransformColumnTypes(#"Naam kolom aanpassen",{{"Date", type date}}),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Jaar ingevoegd" = Table.AddColumn(#"Set type as date", "Jaar", each Date.Year([Date]), Int64.Type),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Maand ingevoegd" = Table.AddColumn(#"Jaar ingevoegd", "MaandNummer", each Date.Month([Date]), Int64.Type),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Week van jaar ingevoegd" = Table.AddColumn(#"Maand ingevoegd", "WeekNummer", each Date.WeekOfYear([Date]), Int64.Type),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Naam van maand ingevoegd" = Table.AddColumn(#"Week van jaar ingevoegd", "Maand", each Date.MonthName([Date]), type text),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Dag van week ingevoegd" = Table.AddColumn(#"Naam van maand ingevoegd", "Dag van week", each Date.DayOfWeek([Date]), Int64.Type),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Ingevoegde naam van de dag" = Table.AddColumn(#"Dag van week ingevoegd", "Naam van de dag", each Date.DayOfWeekName([Date]), type text),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Kwartaal ingevoegd" = Table.AddColumn(#"Ingevoegde naam van de dag", "Kwartaal", each Date.QuarterOfYear([Date]), Int64.Type),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Dag ingevoegd" = Table.AddColumn(#"Kwartaal ingevoegd", "Dag", each Date.Day([Date]), Int64.Type),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Samengevoegde kolom ingevoegd" = Table.AddColumn(#"Dag ingevoegd", "Q Kwartaal", each Text.Combine({"Q", Text.From([Kwartaal], "nl-NL")}), type text),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Sorted Rows" = Table.Sort(#"Samengevoegde kolom ingevoegd",{{"Date", Order.Descending}})<br>
in<br>
&nbsp;&nbsp;&nbsp;&nbsp;    #"Sorted Rows"<br>
<br>
# Counts<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;= Distintcount([waarde])<br>
<br>
# Running total<br>

RunningTotalTrajecten = IF(SELECTEDVALUE(Dim_Date[date]) >= TODAY(),BLANK(),<br>
CALCULATE([Aantal Trajecten],<br>
&nbsp;&nbsp;&nbsp;&nbsp;    FILTER(ALLSELECTED(Dim_Date[Date]),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    Dim_Date[Date] <= MAX(Dim_Date[Date])<br>
&nbsp;&nbsp;&nbsp;&nbsp;    ),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    Dim_Date[Date] <= TODAY(),<br>
&nbsp;&nbsp;&nbsp;&nbsp;    Dim_Date[Jaar] = 2024<br>
))<br>
<br>

# Regressie analyse<br>

Linear regression Trajecten = <br>
VAR Known =<br>
&nbsp;&nbsp;&nbsp;&nbsp;    FILTER (<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        SELECTCOLUMNS (<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            ALLSELECTED ( 'Dim_Date'[Date] ), // Date tabel date column<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            "Known[X]", 'Dim_Date'[Date], <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            "Known[Y]", [RunningTotalTrajecten] // Runing totals<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        ),<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        AND (<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            NOT ( ISBLANK ( Known[X] ) ),<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;            NOT ( ISBLANK ( Known[Y] ) )<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        )<br>
&nbsp;&nbsp;&nbsp;&nbsp;    )<br>
VAR SlopeIntercept =<br>
&nbsp;&nbsp;&nbsp;&nbsp;    LINESTX(Known, Known[Y], Known[X])<br>
VAR Slope =<br>
&nbsp;&nbsp;&nbsp;&nbsp;    SELECTCOLUMNS(SlopeIntercept, [Slope1])<br>
VAR Intercept = <br>
&nbsp;&nbsp;&nbsp;&nbsp;    SELECTCOLUMNS(SlopeIntercept, [Intercept])<br>
RETURN<br>
&nbsp;&nbsp;&nbsp;&nbsp;    SUMX (<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        DISTINCT ( 'Dim_Date'[Date] ),<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        Intercept + Slope * 'Dim_Date'[Date]<br>
&nbsp;&nbsp;&nbsp;&nbsp;    )<br>
