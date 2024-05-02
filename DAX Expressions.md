# Counts
```vbscript 
=
Distintcount([waarde])
```

# Running total
```vbscript 
=
IF(SELECTEDVALUE(Dim_Date[date]) >= TODAY(),BLANK(),
CALCULATE([Aantal Trajecten],
     FILTER(ALLSELECTED(Dim_Date[Date]),
     Dim_Date[Date] <= MAX(Dim_Date[Date])
     ),
     Dim_Date[Date] <= TODAY(),
     Dim_Date[Jaar] = 2024
))
```
# Regressie analyse<br>
```vbscript
=
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
```
# Totals
```vbscript 
=
TOTALYTD()
```
