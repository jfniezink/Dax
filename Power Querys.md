# Power Query items

## Dim Date query <br>
```objectivec
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

# connectors

## sharepoint

```objectivec
// hiervoor moet je wel eigenaar van de site zijn.
// voor persoonlijke onedrive is de url anders, namelijk; https://[domain]-my.sharepoint.com/ 
sharepoint.files(url as text) voor sharepoint files
sharepoint.contents(url as text, [ApiVersion = 15]) voor sharepoint contents.
```

## Rest API AFAS connector
```objectivec
let
    // Vul hier je token
    Token = "",

    // Onderstaande zorgt voor de juiste authorisatie string
    BinaryToken = "AfasToken " & Binary.ToText(Text.ToBinary(Token)),


    // Vul je omgevingsID in onderstaande variabele in tussen ""
    OmgevingsID = "000000",

    // Op basis van soort omgeving veranderd de URL:
    // Productie omgeving = https://000000.rest.afas.online/ProfitRestServices/connectors/
    // Test omgeving = https://000000.resttest.afas.online/ProfitRestServices/connectors/
    // Accept omgeving = https://000000.restaccept.afas.online/ProfitRestServices/connectors/

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
```objectivec
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
        &#60soap:Envelope
            xmlns:xsi=#(0022)http://www.w3.org/2001/XMLSchema-instance#(0022)
            xmlns:xsd=#(0022)http://www.w3.org/2001/XMLSchema#(0022)
            xmlns:soap=#(0022)http://schemas.xmlsoap.org/soap/envelope/#(0022)>
                &#60soap:Body>
                    &#60GetData
                        xmlns=#(0022)urn:Afas.Profit.Services#(0022)>
                        &#60token>
                            &#60![CDATA[" & token & "]]>
                        &#60/token>
                        &#60connectorId>" & ConnectorID & "&#60/connectorId>
                        &#60skip>" & skip & "&#60/skip>
                        &#60take>" & take & "&#60/take>
                        &#60/GetData>
                    &#60/soap:Body>
                &#60/soap:Envelope>
        ",

    options = [
        #"Content-Type"="text/xml;charset=utf-8"
    ],

    Source = Xml.Tables(Web.Contents(url, [Content=Text.ToBinary(SOAPEnvelope), Headers = options]))
in
    Source
```
