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
