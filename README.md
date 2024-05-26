# utl-has-your-stock-out-performed-standard-and-poors500-calulating-the-beta-slope
Has your stock out performed standard and poors500 over the last two years using financial beta index
    %let pgm=utl-has-your-stock-out-performed-standard-and-poors500-calulating-the-beta-slope;

    Has your stock out performed standard and poors 500 over the last two years using financial beta index

    github
    https://tinyurl.com/4arv2suv
    https://github.com/rogerjdeangelis/utl-has-your-stock-out-performed-standard-and-poors500-calulating-the-beta-slope

    stackoverflow
    https://tinyurl.com/bdz6u8fx
    https://stackoverflow.com/questions/32186233/r-calculating-a-stocks-beta-using-performanceanalytics-capm-beta-function-or

    %stop_submission;

    We are using the month end adjusted close for 24 months from 2022-04-01 to 2024-03-01

    The ALTAIR beta below of 1.3 indicates that ALTAIR stock is expected to rise
    1.3% for every 1% rise in the S&P Index.

    Note R provides an inteface of all NY Stock exchange daily stock statistics

      Two solutions

           1 SAS ( R is used to create input) uses stattransfer
           2 R

    SOAPBOX ON

    The problem with R and more so with Python is the dozens of boutique
    data types and data structures.
    You often need to learn a dozen different R and Python functions and statements
    to convert those data structures and data types into something another
    language can use. Note all the code below to change
    the R xts (extensible time series object) into a dataframe that sql and eventually
    SAS can use.
    Column names like ALTR.Adjusted have to be renamed
    Special time series date types  have to be converted to posix
    An odd index type?  removed

    SOAPBOX OFF

    /*               _     _
     _ __  _ __ ___ | |__ | | ___ _ __ ___
    | `_ \| `__/ _ \| `_ \| |/ _ \ `_ ` _ \
    | |_) | | | (_) | |_) | |  __/ | | | | |
    | .__/|_|  \___/|_.__/|_|\___|_| |_| |_|
    |_|
    */

    /**************************************************************************************************************************/
    /*                                   |                                           |                                        */
    /* SAS (R creates input see below)   |                                           |                                        */
    /*                                   |                                           |                                        */
    /* INPUT (below is access to NTSE)   |          PROCESS                          |             OUTPUT                     */
    /* Monthly closes Altair and S&P     |                                           |                                        */
    /*                                   |                                           |                                        */
    /* ROWS  MONYR    ALTR     SPY       | 8Monthly Slopes for S&P and ALTAIR;       |          MODEL    VARIABLE    BETA E   */
    /*                                   |    proc sql;                              |                                        */
    /*  1    2022-03  64.40  437.776     |  create                                   | Slope_S&P_vs_ALTR Intercept  0.00556   */
    /*  2    2022-04  54.32  399.353     |     table want as                         | Slope_S&P_vs_ALTR SPYSLOPE   1.31251   */
    /*  3    2022-05  54.96  400.254     |  select                                   |                                        */
    /*  4    2022-06  52.50  367.249     |     r.monyr  as monyrCur                  |                                        */
    /*  5    2022-07  58.91  401.068     |    ,(r.altr - l.altr)/l.altr as altrSlope |                                        */
    /*  6    2022-08  52.01  384.704     |    ,(r.spy  - l.spy )/l.spy  as spySlope  |                                        */
    /*  7    2022-09  44.22  349.140     |  from                                     |                                        */
    /*  8    2022-10  49.05  377.516     |     tmp.want as l, tmp.want as r          |                                        */
    /*  9    2022-11  49.07  398.503     |  where                                    |                                        */
    /* 10    2022-12  45.47  375.538     |     l.rownames + 1 = r.rownames           |                                        */
    /* 11    2023-01  53.10  399.154     | ;quit;                                    |                                        */
    /* 12    2023-02  64.06  389.119     |                                           |                                        */
    /* 13    2023-03  72.11  403.546     |Obs    MONYRCUR    ALTRSLOPE     SPYSLOPE  |                                        */
    /* 14    2023-04  69.05  409.993     |                                           |                                        */
    /* 15    2023-05  73.33  411.885     |  1    2022-04      -0.15652    -0.087769  |                                        */
    /* 16    2023-06  75.84  438.576     |  2    2022-05       0.01178     0.002257  |                                        */
    /* 17    2023-07  74.94  452.932     |  3    2022-06      -0.04476    -0.082460  |                                        */
    /* 18    2023-08  66.48  445.570     | ...                                       |                                        */
    /* 19    2023-09  62.56  424.435     | 24    2024-03       0.01258     0.032702  |                                        */
    /* 20    2023-10  62.12  415.221     | 25    2024-04      -0.06616    -0.040320  |                                        */
    /* 21    2023-11  72.46  453.149     | 26    2024-05       0.15475     0.054703  |                                        */
    /* 22    2023-12  84.15  473.838     |                                           |                                        */
    /* 23    2024-01  85.02  481.384     | ods trace on;                             |                                        */
    /* 24    2024-02  85.08  506.506     | ods output ParameterEstimates             |                                        */
    /* 25    2024-03  86.15  523.070     |   =coef;                                  |                                        */
    /* 26    2024-04  80.45  501.980     | proc reg data=want(obs=24 );              |                                        */
    /* 27    2024-05  92.90  529.440     |  'Slope_S_and_P_vs_ALTR'n:                |                                        */
    /*                                   |    Model altrSlope=spySlope;              |                                        */
    /*                                   | run;quit;                                 |                                        */
    /*                                   | ods trace off;                            |                                        */
    /*                                   |                                           |                                        */
    /*                                   | proc print data=coef;                     |                                        */
    /*                                   | var model dependent                       |                                        */
    /*                                   |   variable estimate;                      |                                        */
    /*                                   | run;quit;                                 |                                        */
    /*                                   |                                           |                                        */
    /* ----------------------------------|------------------------------------------------------------------------------------*/
    /*                                   |                                           |                                        */
    /*  R                                |                                           |                                        */
    /*                                   |                                           |                                        */
    /*  INPUT EXTRACTED IN R SCRIPT      |            PROCESS                        |                OUTPUT                  */
    /*                                   |                                           |                                        */
    /*                                   |  %utl_submit_r64('                        |      (Intercept) r(spy)[2:25]          */
    /*                                   |  library(PerformanceAnalytics);           |      0.005560156  1.312510904          */
    /*                                   |  library(quantmod);                       |                                        */
    /*                                   |  start_date <- "2022-03-01";              |                                        */
    /*                                   |  altr<-getSymbols("ALTR"                  |                                        */
    /*                                   |  ,from=start_date,auto.assign=F);         |                                        */
    /*                                   |  spy<-getSymbols("SPY",from=start_date    |                                        */
    /*                                   |  ,auto.assign=F);                         |                                        */
    /*                                   |  r<-function(x)                           |                                        */
    /*                                   |    {m<-to.monthly(x[,6])[,4];             |                                        */
    /*                                   |    diff(m)/lag(m)                         |                                        */
    /*                                   |    };                                     |                                        */
    /*                                   |  ac<-as.data.frame(r(altr)[2:25]);        |                                        */
    /*                                   |  sp<-as.data.frame(r(spy)[2:25]);         |                                        */
    /*                                   |  coef(lm(r(altr)[2:25]~r(spy)[2:25]));    |                                        */
    /*                                   |  ');                                      |                                        */
    /*                                   |                                           |                                        */
    /**************************************************************************************************************************/

    /*
    / |  ___  __ _ ___
    | | / __|/ _` / __|
    | | \__ \ (_| \__ \
    |_| |___/\__,_|___/
     _                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */


    %utl_rbegin
    parmcards4
    library(PerformanceAnalytics)
    library(quantmod)
    library(sqldf)
    source("c:/temp/fn_tosas9.R")
    start_date <- "2022-03-01"
    altr<-getSymbols("ALTR",from=start_date,auto.assign=F)
    altr <- cbind(as.data.frame(altr), Date = index(altr))
    altr$altr_close<- altr$ALTR.Close;
    altr$altr_adjusted<- altr$ALTR.Adjusted;
    altr$monyr<- as.character(format(altr$Date, "%Y-%m"))
    altr$datef<-as.character(format(altr$Date,"%Y-%m-%d"))
    altr<-sqldf('
      select
         datef
        ,monyr
        ,altr_adjusted as altr
      from
        altr
      group
        by monyr
      having
        date = max(date)
      ')
    spy<-getSymbols("SPY",from=start_date,auto.assign=F)
    spy <- cbind(as.data.frame(spy), Date = index(spy))
    spy$spy_close<- spy$SPY.Close;
    spy$spy_adjusted<- spy$SPY.Adjusted;
    spy$monyr<- as.character(format(spy$Date, "%Y-%m"))
    spy$datef<-as.character(format(spy$Date,"%Y-%m-%d"))

    spy<-sqldf('
      select
         datef
        ,monyr
        ,spy_adjusted as spy
      from
        spy
      group
        by monyr
      having
        date = max(date)
      ')
    want<-sqldf('
       select
          l.monyr
         ,l.altr
         ,r.spy
       from
          altr as l left join spy as r
       on
          l.monyr = r.monyr
       ')
    fn_tosas9(dataf=want);
    ;;;;
    %utl_rend;

    libname tmp "c:/temp";
    proc print data=tmp.want;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* c:/temp/want.sas7bdat                                                                                                  */
    /*                                                                                                                        */
    /* ROWNAMES     MONYR       ALTR       SPY                                                                                */
    /*                                                                                                                        */
    /*     1       2022-03    64.4000    437.776                                                                              */
    /*     2       2022-04    54.3200    399.353                                                                              */
    /*     3       2022-05    54.9600    400.254                                                                              */
    /*     4       2022-06    52.5000    367.249                                                                              */
    /*     5       2022-07    58.9100    401.068                                                                              */
    /*     6       2022-08    52.0100    384.704                                                                              */
    /*     7       2022-09    44.2200    349.140                                                                              */
    /*     8       2022-10    49.0500    377.516                                                                              */
    /*     9       2022-11    49.0700    398.503                                                                              */
    /*    10       2022-12    45.4700    375.538                                                                              */
    /*    11       2023-01    53.1000    399.154                                                                              */
    /*    12       2023-02    64.0600    389.119                                                                              */
    /*    13       2023-03    72.1100    403.546                                                                              */
    /*    14       2023-04    69.0500    409.993                                                                              */
    /*    15       2023-05    73.3300    411.885                                                                              */
    /*    16       2023-06    75.8400    438.576                                                                              */
    /*    17       2023-07    74.9400    452.932                                                                              */
    /*    18       2023-08    66.4800    445.570                                                                              */
    /*    19       2023-09    62.5600    424.435                                                                              */
    /*    20       2023-10    62.1200    415.221                                                                              */
    /*    21       2023-11    72.4600    453.149                                                                              */
    /*    22       2023-12    84.1500    473.838                                                                              */
    /*    23       2024-01    85.0200    481.384                                                                              */
    /*    24       2024-02    85.0800    506.506                                                                              */
    /*    25       2024-03    86.1500    523.070                                                                              */
    /*    26       2024-04    80.4500    501.980                                                                              */
    /*    27       2024-05    92.9000    529.440                                                                              */
    /*                                                                                                                        */
    /* INTERMEDIATE DATASET for S&P 500                                                                                       */
    /*                                                                                                                        */
    /*            SPY.Open SPY.High SPY.Low SPY.Close SPY.Volume SPY.Adjusted                                                 */
    /*                                                                                                                        */
    /* 2022-03-01   435.04   437.17  427.11    429.98  137785900     415.4902                                                 */
    /* 2022-03-02   432.37   439.72  431.57    437.89  117726500     423.1336                                                 */
    /* 2022-03-03   440.47   441.11  433.80    435.71  105501700     421.0271                                                 */
    /* 2022-03-04   431.75   433.37  427.88    432.17  113978200     417.6065                                                 */
    /* 2022-03-07   431.55   432.30  419.36    419.43  137896600     405.2957                                                 */
    /* 2022-03-08   419.62   427.21  415.12    416.25  164772700     402.2229                                                 */
    /* 2022-03-09   425.14   429.51  422.82    427.41  116990800     413.0068                                                 */
    /* 2022-03-10   422.52   426.43  420.44    425.48   93972700     411.1418                                                 */
    /* 2022-03-11   428.12   428.77  419.53    420.07   95636300     405.9142                                                 */
    /* 2022-03-14   420.89   424.55  415.79    417.00   95729200     402.9477                                                 */
    /* 2022-03-15   419.77   426.84  418.42    426.17  106219100     411.8086                                                 */
    /* ...                                                                                                                    */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*
     _ __  _ __ ___   ___ ___  ___ ___
    | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    */

     proc sql;
      create
         table want as
      select
         r.monyr  as monyrCur
        ,(r.altr - l.altr)/l.altr as altrSlope
        ,(r.spy  - l.spy )/l.spy  as spySlope
      from
         tmp.want as l, tmp.want as r
      where
         l.rownames + 1 = r.rownames
    ;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* Obs    MONYRCUR    ALTRSLOPE     SPYSLOPE                                                                              */
    /*                                                                                                                        */
    /*   1    2022-04      -0.15652    -0.087769                                                                              */
    /*   2    2022-05       0.01178     0.002257                                                                              */
    /*   3    2022-06      -0.04476    -0.082460                                                                              */
    /*  ...                                                                                                                   */
    /*  24    2024-03       0.01258     0.032702                                                                              */
    /*  25    2024-04      -0.06616    -0.040320                                                                              */
    /*  26    2024-05       0.15475     0.054703                                                                              */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

     ods trace on;
     ods output ParameterEstimates
       =coef;
     proc reg data=want(obs=24 );
      'Slope_S_and_P_vs_ALTR'n:
        Model altrSlope=spySlope;
     run;quit;
     ods trace off;

     proc print data=coef;
     var model dependent
       variable estimate;
     run;quit;

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* COEF total obs=2                                                                                                       */
    /* Obs            MODEL            DEPENDENT    VARIABLE     DF    ESTIMATE     STDERR     TVALUE     PROBT               */
    /*                                                                                                                        */
    /*  1     Slope_S_and_P_vs_ALTR    ALTRSLOPE    Intercept     1     0.00556    0.01561    0.35614    0.72513              */
    /*  2     Slope_S_and_P_vs_ALTR    ALTRSLOPE    SPYSLOPE      1     1.31251    0.27893    4.70544    0.00011              */
    /*                                                                                                                        */
    /* PROC PRINT                                                                                                             */
    /*                                                                                                                        */
    /*         MODEL            DEPENDENT    VARIABLE        ESTIMATE                                                         */
    /*                                                                                                                        */
    /* Slope_S_and_P_vs_ALTR    ALTRSLOPE    Intercept        0.00556                                                         */
    /* Slope_S_and_P_vs_ALTR    ALTRSLOPE    SPYSLOPE         1.31251                                                         */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*___
    |___ \   _ __
      __) | | `__|
     / __/  | |
    |_____| |_|

    */

    %utl_submit_r64('
    library(PerformanceAnalytics);
    library(quantmod);
    start_date <- "2022-03-01";
    altr<-getSymbols("ALTR"
    ,from=start_date,auto.assign=F);
    spy<-getSymbols("SPY",from=start_date
    ,auto.assign=F);
    r<-function(x)
      {m<-to.monthly(x[,6])[,4];
      diff(m)/lag(m)
      };
    ac<-as.data.frame(r(altr)[2:25]);
    sp<-as.data.frame(r(spy)[2:25]);
    coef(lm(r(altr)[2:25]~r(spy)[2:25]));
    ');

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  (Intercept) r(spy)[2:25]                                                                                              */
    /*   0.00556015   1.31251070                                                                                              */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
