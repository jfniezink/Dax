# Power Query items

## Dim Date query <br>

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

# connectors

## sharepoint

// hiervoor moet je wel eigenaar van de site zijn. <br>
// voor persoonlijke onedrive is de url anders, namelijk; https://[domain]-my.sharepoint.com/ <br>
sharepoint.files(url as text) voor sharepoint files<br>
sharepoint.contents(url as text, [ApiVersion = 15]) voor sharepoint contents. <br>

## Rest API AFAS connector

let<br>
&nbsp;&nbsp;&nbsp;&nbsp;  // Vul hier je token tussen de "" markering<br>
&nbsp;&nbsp;&nbsp;&nbsp;  Token = "Token",<br>
&nbsp;&nbsp;&nbsp;&nbsp;  // Onderstaande zorgt voor de juiste authorisatie string<br>
&nbsp;&nbsp;&nbsp;&nbsp;  BinaryToken = "AfasToken " & Binary.ToText(Text.ToBinary(Token)),<br>
&nbsp;&nbsp;&nbsp;&nbsp;  // Vul in onderstaande de URL voor verbinding in<br>
&nbsp;&nbsp;&nbsp;&nbsp;  // Op basis van soort omgeving veranderd de URL:<br>
&nbsp;&nbsp;&nbsp;&nbsp;  // Productie omgeving = https://000000.rest.afas.online/ProfitRestServices/connectors/<br>
&nbsp;&nbsp;&nbsp;&nbsp;  // Test omgeving = https://000000.resttest.afas.online/ProfitRestServices/connectors/<br>
&nbsp;&nbsp;&nbsp;&nbsp;  // Accept omgeving = https://000000.restaccept.afas.online/ProfitRestServices/connectors/<br>
&nbsp;&nbsp;&nbsp;&nbsp;  URL = "https://000000.rest.afas.online/ProfitRestServices/connectors/",<br>
&nbsp;&nbsp;&nbsp;&nbsp;  // Vul bij connector het ID van je connector in. Zorg dat je rechten hebt om deze aan te spreken. <br>
&nbsp;&nbsp;&nbsp;&nbsp;  Connector = test_connector,<br>
&nbsp;&nbsp;&nbsp;&nbsp;  // Vul het aantal regels dat geskipt moet worden in onderstaande waarde<br>
&nbsp;&nbsp;&nbsp;&nbsp;  skipvalue = -1,<br>
&nbsp;&nbsp;&nbsp;&nbsp;  Skip = "?skip=" & skipvalue,<br>
&nbsp;&nbsp;&nbsp;&nbsp;  // Vul het aantal regels dat opgehaald moet worden in onderstaande waarde<br>
&nbsp;&nbsp;&nbsp;&nbsp;  takevalue = -1,<br>
&nbsp;&nbsp;&nbsp;&nbsp;  Take = "?take=" & takevalue,<br>
&nbsp;&nbsp;&nbsp;&nbsp;  Source = <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Json.Document(<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Web.Contents(<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;URL & <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Connector & <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Skip & <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Take, <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Headers=<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Authorization=BinaryToken<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;]<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;]<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br>
in<br>
&nbsp;&nbsp;&nbsp;&nbsp; Source<br>
