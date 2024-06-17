# Power Query items

## Dim Date query Short
```vbscript
let
     // Noem je datum tabel: Dim_Date
     // Variabelen
     Startdatum = #date(2010, 1, 1),
     Duur = Duration.Days(Date.From(Date.EndOfYear(DateTime.LocalNow()))-Startdatum)+1,
     //Datum tabel maken
     #"Lijst maken" = List.Dates(Startdatum,Duur,#duration(1,0,0,0)),
     #"Tabel maken" = Table.FromList(#"Lijst maken", Splitter.SplitByNothing(), null, null, ExtraValues.Error),
     #"Naam kolom aanpassen" = Table.RenameColumns(#"Tabel maken",{{"Column1", "Date"}}),
     #"Set type as date" = Table.TransformColumnTypes(#"Naam kolom aanpassen",{{"Date", type date}}),
     #"Jaar ingevoegd" = Table.AddColumn(#"Set type as date", "Jaar", each Date.Year([Date]), Int64.Type),
     #"Maand ingevoegd" = Table.AddColumn(#"Jaar ingevoegd", "MaandNummer", each Date.Month([Date]), Int64.Type),
     #"Week van jaar ingevoegd" = Table.AddColumn(#"Maand ingevoegd", "WeekNummer", each Date.WeekOfYear([Date]), Int64.Type),
     #"Naam van maand ingevoegd" = Table.AddColumn(#"Week van jaar ingevoegd", "Maand", each Date.MonthName([Date]), type text),
     #"Dag van week ingevoegd" = Table.AddColumn(#"Naam van maand ingevoegd", "Dag van week", each Date.DayOfWeek([Date]), Int64.Type),
     #"Ingevoegde naam van de dag" = Table.AddColumn(#"Dag van week ingevoegd", "Naam van de dag", each Date.DayOfWeekName([Date]), type text),
     #"Kwartaal ingevoegd" = Table.AddColumn(#"Ingevoegde naam van de dag", "Kwartaal", each Date.QuarterOfYear([Date]), Int64.Type),
     #"Dag ingevoegd" = Table.AddColumn(#"Kwartaal ingevoegd", "Dag", each Date.Day([Date]), Int64.Type),
     #"Samengevoegde kolom ingevoegd" = Table.AddColumn(#"Dag ingevoegd", "Q Kwartaal", each Text.Combine({"Q", Text.From([Kwartaal], "nl-NL")}), type text),
     #"Sorted Rows" = Table.Sort(#"Samengevoegde kolom ingevoegd",{{"Date", Order.Descending}})
in
     #"Sorted Rows"
```

# Dim Date Query Expanded with ISO week convertor
```vbscript
let
    Source = List.Dates(StartDate, Length, #duration(1, 0, 0, 0)),
    // Vul het startjaar en eindjaar
    Start_year_calendar = 2018,
    End_year_calendar = 2024,
    StartDate = #date(Start_year_calendar,1, 1),
    EndDate = #date(End_year_calendar, 12, 31) ,
    Today = DateTime.Date(DateTime.LocalNow()),
    Length = Duration.Days(EndDate - StartDate) + 1,
    DayOfWeek = Date.DayOfWeek(Today)+1,
    StartWeekDate = Date.AddDays(Today, 1-DayOfWeek),
    DayOfMonth = Date.Day(Today),
    StartMonthDate = Date.AddDays(Today,1-DayOfMonth),
    EndQuarterDate = Date.EndOfQuarter(Today),

    getISO8601Week = (someDate as date) =>
        let
            getDayOfWeek = (d as date) =>
                let
                    result = 1 + Date.DayOfWeek(d, Day.Monday)
                in
                    result,

            getNaiveWeek = (inDate as date) =>
                let
                    // monday = 1, sunday = 7
                    weekday = getDayOfWeek(inDate),

                    weekdayOfJan4th = getDayOfWeek(#date(Date.Year(inDate), 1, 4)),

                    ordinal = Date.DayOfYear(inDate),

                    naiveWeek = Number.RoundDown(
                        (ordinal - weekday + 10) / 7
                    )
                in
                    naiveWeek,

            thisYear = Date.Year(someDate),

            priorYear = thisYear - 1,

            nwn = getNaiveWeek(someDate),

            lastWeekOfPriorYear =
                getNaiveWeek(#date(priorYear, 12, 28)),

            // http://stackoverflow.com/a/34092382/2014893
            lastWeekOfThisYear =
                getNaiveWeek(#date(thisYear, 12, 28)),

            weekYear =
                if
                    nwn < 1
                then
                    priorYear
                else
                    if
                        nwn > lastWeekOfThisYear
                    then
                        thisYear + 1
                    else
                        thisYear,

            weekNumber =
                if
                    nwn < 1
                then
                    lastWeekOfPriorYear
                else
                    if
                        nwn > lastWeekOfThisYear
                    then
                        1
                    else
                        nwn,

            week_dateString =
                Text.PadStart(
                    Text.From(
                        Number.RoundDown(weekNumber)
                    ),
                    2,
                    "0"
                )
        in
            Text.From(weekYear) & "-W" & week_dateString & "-" & Text.From(getDayOfWeek(someDate)),


    #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Renamed Columns" = Table.RenameColumns(#"Converted to Table",{{"Column1", "Date"}}),
    #"Changed Type" = Table.TransformColumnTypes(#"Renamed Columns",{{"Date", type date}}),
    Custom1 = #"Changed Type",
    #"Inserted Year" = Table.AddColumn(Custom1, "Year", each Date.Year([Date])),
    #"Inserted Month" = Table.AddColumn(#"Inserted Year", "Month", each Date.Month([Date])),
    #"Inserted Quarter" = Table.AddColumn(#"Inserted Month", "Quarter", each Date.QuarterOfYear([Date])),
    #"Inserted Day" = Table.AddColumn(#"Inserted Quarter", "Day", each Date.Day([Date])),
    #"Inserted Quarter Name" = Table.AddColumn(#"Inserted Day", "Quarter Name", each Text.Combine({"Q",Number.ToText([Quarter])})),
    #"Inserted YYYYQ" = Table.AddColumn(#"Inserted Quarter Name", "YYYYQ", each [Year] * 10 + [Quarter]),
    #"Inserted Quarter End Date" = Table.AddColumn(#"Inserted YYYYQ", "Quarter Date End", each Date.EndOfQuarter([Date])),
    #"Inserted Quarter Start Date" = Table.AddColumn(#"Inserted Quarter End Date", "Quarter Date Start", each Date.AddDays(Date.EndOfMonth(Date.AddQuarters([Quarter Date End],-1)),1)),
    #"Inserted Month Name" = Table.AddColumn(#"Inserted Quarter Start Date", "Month Name", each Date.MonthName([Date], "en-US"), type text),
    #"Inserted Month Name NL" = Table.AddColumn(#"Inserted Month Name", "Month Name NL", each Date.MonthName([Date], "nl-NL")),
    #"Inserted DateKey" = Table.AddColumn(#"Inserted Month Name NL", "DateKey", each [Year] * 10000 + [Month] * 100 + [Day]),
    #"Reordered Columns" = Table.ReorderColumns(#"Inserted DateKey",{"DateKey", "Date", "Year", "Quarter", "Month", "Quarter Name", "YYYYQ", "Quarter Date Start", "Quarter Date End", "Day", "Month Name"}),
    #"Inserted Month Name Short" = Table.AddColumn(#"Reordered Columns", "Month Name Short", each Text.Start([Month Name], 3), type text),
    #"Inserted Month Name Short NL" = Table.AddColumn(#"Inserted Month Name Short", "Month Name Short NL", each Date.ToText([Date], "MMM")),
    #"Inserted YYYYMM" = Table.AddColumn(#"Inserted Month Name Short NL", "YYYYMM", each [Year]*100 + [#"Month"]),
    #"Inserted MMM-YYYY" = Table.AddColumn(#"Inserted YYYYMM", "MMM-YYYY", each Text.Combine({[Month Name Short], "-", Number.ToText([Year])})),
    #"Inserted MMM-YYYY NL" = Table.AddColumn(#"Inserted MMM-YYYY", "MMM-YYYY NL", each Text.Combine({[Month Name Short NL], "-", Number.ToText([Year])})),
    #"Inserted Month Start Date" = Table.AddColumn(#"Inserted MMM-YYYY NL", "Month Date Start", each Date.AddDays([Date], 1-[Day])),
    #"Inserted Month End Date" = Table.AddColumn(#"Inserted Month Start Date", "Month Date End", each Date.EndOfMonth([Month Date Start])),
    #"Inserted Day of Week" = Table.AddColumn(#"Inserted Month End Date", "Day of Week", each Date.DayOfWeek([Date])+1, Int64.Type),
    #"Inserted Day Name" = Table.AddColumn(#"Inserted Day of Week", "Day Name", each Date.DayOfWeekName([Date], "en-US"), type text),
    #"Inserted Day Name NL" = Table.AddColumn(#"Inserted Day Name", "Day Name NL", each Date.DayOfWeekName([Date], "nl-NL")),
    #"Inserted Day Name Short" = Table.AddColumn(#"Inserted Day Name NL", "Day Name Short", each Text.Start([Day Name], 3), type text),
    #"Inserted Day Name Short NL" = Table.AddColumn(#"Inserted Day Name Short", "Day Name Short NL", each Text.Start([Day Name NL], 2)),
    #"Inserted ISO Date" = Table.AddColumn(#"Inserted Day Name Short NL", "getISODate", each getISO8601Week([Date])),
    #"Inserted ISO Year" = Table.AddColumn(#"Inserted ISO Date", "ISO Year", each Text.Start([getISODate], 4)),
    #"Inserted ISO Week" = Table.AddColumn(#"Inserted ISO Year", "ISO Week", each Text.Middle([getISODate], 6, 2)),
    #"Inserted ISO Week Name" = Table.AddColumn(#"Inserted ISO Week", "ISO Week Name", each Text.Middle([getISODate], 5, 3)),
    #"Changed Type Numeric" = Table.TransformColumnTypes(#"Inserted ISO Week Name",{{"Year", Int64.Type}, {"DateKey", Int64.Type}, {"Quarter", Int64.Type}, {"Month", Int64.Type}, {"YYYYQ", Int64.Type}, {"Day", Int64.Type}, {"YYYYMM", Int64.Type}, {"Day of Week", Int64.Type}, {"ISO Year", Int64.Type}, {"ISO Week", Int64.Type}}),
    #"Inserted YYYWW" = Table.AddColumn(#"Changed Type Numeric", "YYYYWW", each [ISO Year] * 100 + [ISO Week]),
    #"Inserted Week Start Date" = Table.AddColumn(#"Inserted YYYWW", "Week Date Start", each Date.AddDays([Date], 1-[Day of Week])),
    #"Inserted Week End Date" = Table.AddColumn(#"Inserted Week Start Date", "Week Date End", each Date.AddDays([Week Date Start], 6)),
    #"Inserted Day Index" = Table.AddColumn(#"Inserted Week End Date", "Day Index", each Duration.Days(Duration.From([Date] - Today))),
    #"Inserted Week Index" = Table.AddColumn(#"Inserted Day Index", "Week Index", each Duration.Days(Duration.From(StartWeekDate - [Week Date Start])) / -7),
    #"Inserted Month Index" = Table.AddColumn(#"Inserted Week Index", "Month Index", each Number.Round(Duration.Days(Duration.From(StartMonthDate - [Month Date Start])) / -30.4 , 0)),
    #"Inserted Quarter Index" = Table.AddColumn(#"Inserted Month Index", "Quarter Index", each Number.Round(Duration.Days(Duration.From(EndQuarterDate - [Quarter Date End])) / -91.25 , 0)),
    #"Inserted Year Index" = Table.AddColumn(#"Inserted Quarter Index", "Year Index", each [Year] - Date.Year(Today)),
    #"Changed Type Date" = Table.TransformColumnTypes(#"Inserted Year Index",{{"YYYYWW", Int64.Type}, {"Week Index", Int64.Type}, {"Month Index", Int64.Type}, {"Year Index", Int64.Type}, {"Week Date Start", type date}, {"Week Date End", type date}, {"Month Date End", type date}, {"Month Date Start", type date}, {"getISODate", type text}, {"ISO Week Name", type text}, {"Quarter Date Start", type date}, {"Quarter Date End", type date}, {"Quarter Index", Int64.Type}, {"Day Index", Int64.Type}, {"Day Name Short NL", type text}, {"Day Name NL", type text}, {"MMM-YYYY", type text}, {"MMM-YYYY NL", type text}, {"Month Name NL", type text}}),
    #"Inserted Current Week" = Table.AddColumn(#"Changed Type Date", "Current Week", each if[Week Index] = 0 then "Current Week" else Text.Combine({[ISO Week Name], "-", Number.ToText([Year])})),
    #"Inserted Current Month" = Table.AddColumn(#"Inserted Current Week", "Current Month", each if [Month Index] = 0 then "Current Month" else [#"MMM-YYYY"]),
    #"Inserted Current Quarter" = Table.AddColumn(#"Inserted Current Month", "Current Quarter", each if [Quarter Index] = 0 then "Current Quarter" else Text.Combine({[Quarter Name], "-", Number.ToText([Year])})),
    #"Inserted Current Year" = Table.AddColumn(#"Inserted Current Quarter", "Current Year", each if [Year Index] = 0 then "Current Year" else Number.ToText([Year])),
    #"Inserted Day Type" = Table.AddColumn(#"Inserted Current Year", "Day Type", each if [Day of Week] <= 5 then "Weekday" else "Weekend")
in
    #"Inserted Day Type"
```

# Week Converter to ISO 8601
```vbscript
let
    getISO8601Week = (someDate as date) =>
        let
            getDayOfWeek = (d as date) =>
                let
                    result = 1 + Date.DayOfWeek(d, Day.Monday)
                in
                    result,

            getNaiveWeek = (inDate as date) =>
                let
                    // monday = 1, sunday = 7
                    weekday = getDayOfWeek(inDate),

                    weekdayOfJan4th = getDayOfWeek(#date(Date.Year(inDate), 1, 4)),

                    ordinal = Date.DayOfYear(inDate),

                    naiveWeek = Number.RoundDown(
                        (ordinal - weekday + 10) / 7
                    )
                in
                    naiveWeek,

            thisYear = Date.Year(someDate),

            priorYear = thisYear - 1,

            nwn = getNaiveWeek(someDate),

            lastWeekOfPriorYear =
                getNaiveWeek(#date(priorYear, 12, 28)),

            // http://stackoverflow.com/a/34092382/2014893
            lastWeekOfThisYear =
                getNaiveWeek(#date(thisYear, 12, 28)),

            weekYear =
                if
                    nwn < 1
                then
                    priorYear
                else
                    if
                        nwn > lastWeekOfThisYear
                    then
                        thisYear + 1
                    else
                        thisYear,

            weekNumber =
                if
                    nwn < 1
                then
                    lastWeekOfPriorYear
                else
                    if
                        nwn > lastWeekOfThisYear
                    then
                        1
                    else
                        nwn,

            week_dateString =
                Text.PadStart(
                    Text.From(
                        Number.RoundDown(weekNumber)
                    ),
                    2,
                    "0"
                )
        in
            Text.From(weekYear) & "-W" & week_dateString & "-" & Text.From(getDayOfWeek(someDate))
in
    getISO8601Week
```
# connectors

## VisualCrossing weather data

```vbscript
let
    // Let op dat er maar maximaal 1000 records per dag opgehaald kunnen worden. Anders komt een Http 429 error
    // https://www.visualcrossing.com/
    forecast_days = 14, // max 14 days!
    startdatum = "2023-01-01", // vul de datum als tekst in
    einddatum = // datum van vandaag wordt vanzelf erin gezet. 
        Text.Format(
            "#{0}-#{1}-#{2}", 
                {   Text.From(Date.Year(Date.AddDays(DateTime.LocalNow(),forecast_days))),
                    if Date.Month(Date.AddDays(DateTime.LocalNow(),14)) < 10 then "0" & Text.From(Date.Month(Date.AddDays(DateTime.LocalNow(),14))) else Text.From(Date.Month(Date.AddDays(DateTime.LocalNow(),14))), 
                    if Date.Day(Date.AddDays(DateTime.LocalNow(),14)) < 10 then "0" & Text.From(Date.Day(Date.AddDays(DateTime.LocalNow(),14))) else Text.From(Date.Day(Date.AddDays(DateTime.LocalNow(),14)))
                }
        ),
    APIKey = "00", //Enter Api Key from Visual Crossing
    Bron = Json.Document(Web.Contents(
        "https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/Nederland/" 
        & startdatum & "/" & einddatum & 
        "?unitGroup=metric&include=days&key=" & APIKey & "&contentType=json")),
    days = Table.FromRecords( Bron[days] ),
    #"Rijen gesorteerd" = Table.Sort(days,{{"datetime", Order.Descending}}),
    #"Type gewijzigd" = Table.TransformColumnTypes(#"Rijen gesorteerd",{{"datetime", type date}, {"datetimeEpoch", Int64.Type}, {"tempmax", type number}, {"tempmin", type number}, {"temp", type number}, {"feelslikemax", type number}, {"feelslikemin", type number}, {"feelslike", type number}, {"dew", type number}, {"humidity", type number}, {"precip", type number}, {"precipprob", type number}, {"precipcover", type number}, {"preciptype", type any}, {"snow", Int64.Type}, {"snowdepth", Int64.Type}, {"windgust", type number}, {"windspeed", type number}, {"winddir", type number}, {"pressure", type number}, {"cloudcover", type number}, {"visibility", type number}, {"solarradiation", type number}, {"solarenergy", type number}, {"uvindex", Int64.Type}, {"severerisk", Int64.Type}, {"sunrise", type time}, {"sunriseEpoch", Int64.Type}, {"sunset", type time}, {"sunsetEpoch", Int64.Type}, {"moonphase", type number}, {"conditions", type text}, {"description", type text}, {"icon", type text}, {"stations", type any}, {"source", type text}})
in
    #"Type gewijzigd"
```

## sharepoint

```vbscript
// hiervoor moet je wel eigenaar van de site zijn.
// voor persoonlijke onedrive is de url anders, namelijk; https://[domain]-my.sharepoint.com/ 
sharepoint.files(url as text) voor sharepoint files
sharepoint.contents(url as text, [ApiVersion = 15]) voor sharepoint contents.
```

## Rest API AFAS connector
```vbscript
let
    // Vul hier je token
    Token = "",

    // Onderstaande zorgt voor de juiste authorisatie string
    BinaryToken = "AfasToken " & Binary.ToText(Text.ToBinary(Token)),


    // Vul je omgevingsID in onderstaande variabele in tussen ""
    OmgevingsID = "000000",

    // Op basis van soort omgeving veranderd de URL:
    // Productie omgeving = "https://000000.rest.afas.online/ProfitRestServices/connectors/"
    // Test omgeving = "https://000000.resttest.afas.online/ProfitRestServices/connectors/"
    // Accept omgeving = "https://000000.restaccept.afas.online/ProfitRestServices/connectors/"

    URL = "https://"& OmgevingsID &".rest.afas.online/ProfitRestServices/connectors/",

    // Vul bij connector het ID van je connector in. Zorg dat je rechten hebt om deze aan te spreken.
    Connector = "",

    // Vul het aantal regels dat geskipt moet worden in onderstaande waarde
    skipvalue = "-1",

    // Onderstaande variabele zorgt voor de juiste request variabele in de url
    Skip = "?skip=" & skipvalue,

    // Vul het aantal regels dat opgehaald moet worden in onderstaande waarde
    takevalue = "-1",

    // Onderstaande variabele zorgt voor de juiste request variabele in de url
    Take = "?take=" & takevalue,

    Source =
        Json.Document(
            Web.Contents(
                URL &
                Connector &
                Skip &
                Take,
                    [
                        Headers=
                            [
                                Authorization=BinaryToken
                            ]
                        ]
                    )
                )
in
    Source
```

## SOAP API AFAS connector
```vbscript
let
    // Deze Query kan gebruikt worden om via SOAP Api (xml output) AFAS get connectoren aan te spreken.
    // Parameters
    url = "[url]", // vervang [url] door je eigen SOAP URL
    token = "", // vul hier je eigen afas token tussen quotes
    skip = "-1", // vul hier het aantal dat geskipt moet worden tussen quotes. -1 voor alles
    take = "-1", // vul hier het aantal dat opgehaald moet worden tussen quotes. -1 voor alles
    ConnectorID = "", // vul hier de connector ID dat aangesproken moet worden tussen quotes

    SOAPEnvelope =
        "
        <soap:Envelope
            xmlns:xsi=#(0022)http://www.w3.org/2001/XMLSchema-instance#(0022)
            xmlns:xsd=#(0022)http://www.w3.org/2001/XMLSchema#(0022)
            xmlns:soap=#(0022)http://schemas.xmlsoap.org/soap/envelope/#(0022)>
                <soap:Body>
                    <GetData
                        xmlns=#(0022)urn:Afas.Profit.Services#(0022)>
                        <token>
                            <![CDATA[" & token & "]]>
                        </token>
                        <connectorId>" & ConnectorID & "</connectorId>
                        <skip>" & skip & "</skip>
                        <take>" & take & "</take>
                        </GetData>
                    </soap:Body>
                </soap:Envelope>
        ",

    options = [
        #"Content-Type"="text/xml;charset=utf-8"
    ],

    Source = Xml.Tables(Web.Contents(url, [Content=Text.ToBinary(SOAPEnvelope), Headers = options]))
in
    Source
```
