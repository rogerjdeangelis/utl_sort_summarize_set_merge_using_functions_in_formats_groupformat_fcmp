# utl_sort_summarize_set_merge_using_functions_in_formats_groupformat_fcmp
Sort summarize set merge using functions in formats groupformat fcmp.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.


    SAS-L: Sort summarize set merge using functions in formats groupformat fcmp

      github
      https://tinyurl.com/ybq56yx2
      https://github.com/rogerjdeangelis/utl_sort_summarize_set_merge_using_functions_in_formats_groupformat_fcmp

      Note 'proc sql' can sort on functions of variables.

      This has ramificatons beyond this dumb example, functons of multiple variables?

      Seens to be some bugs, functions in formats does not honor GROUPFORMAT

      Non SQL uses of functions

       1. sort by substr(hgtAgeHgt,4,2)
       2. summarize by substr(hgtAgeHgt,4,2)
       3. sort-merge by substr(hgtAgeHgt,4,2)  (BUG?)
       4. datastep by processing  (BUG?)
       5. proc freq  (BUG?)
       6. proc print honors format
       7. proc corresp
       8. proc transpose (ok nice)

    INPUT
    =====

    sort, summarize, set, merge, .. by age when age is imbedded in a string


    WORK.HAVONE total obs=1,000

     Obs       Height Age Weight    BMI

       1          63-52-089         22   * note age =52
       2          83-36-088         25   * note age =36
       3          18-46-041         27
       4          41-38-282         27
       5          35-59-029         23
       6          79-58-214         28
     ...
    1000          79-44-214         28

    WORK.HAVTWO total obs=10

     Obs    HGTAGEWGT    BMI

       1    63-52-089     22
       2    83-36-088     25
       3    18-46-041     27
       4    41-38-282     27
       5    35-59-029     23
       6    79-58-214     28
       7    68-39-070     27
       8    04-47-066     20
       9    76-32-298     24
      10    04-37-206     29


    PROCESS
    =======

      * code function;
      options cmplib = (work.functions);
      proc fcmp outlib=work.functions.cuthgtAgeWgt;
         function cuthgtAgeWgt(hgtAgeWgt $) $11;
             length hgtAgeWgtcut $2;
             hgtAgeWgtcut=substr(hgtAgeWgt,4,2);
         return(hgtAgeWgtcut);
         endsub;
      run;quit;

      * put function in format;
      proc format;
        value $hgtAgeWgt (default=10) other=[cuthgtAgeWgt()];
      run;quit;

      * add fromat to input datasets;
      proc datasets lib=work;
       modify havone;
       format hgtAgeWgt $hgtAgeWgt.;
       modify havtwo;
       format hgtAgeWgt $hgtAgeWgt.;
      run;quit;


    OUTPUT
    ======

      1. sort by substr(hgtAgeHgt,4,2)

         proc report data=havOne out=wantSrt nowd;
           col hgtAgeWgt bmi;
           define hgtAgeWgt/order order=formated format= $hgtAgeWgt.;
         run;quit;

         WORK.WANTSRT

           HGTAGEWGT    BMI

           05-30-206     29
           05-30-206     20
           05-30-206     23
           04-31-143     21
           04-31-143     29
           04-31-143     28
           04-31-143     28
           04-31-143     23

      2. Summarize by substr(hgtAgeHgt,4,2)

          proc summary data=have nway;
          class hgtAgeWgt;
          format hgtAgeWgt $hgtAgeWgt.;
          var bmi;
          output out=wantSmy(drop=_:) mean=avgBMI n=BMI_n;
          run;quit;

          work.wantSmy total obs=29

          bs    HGTAGEWGT     AVGBMI    BMI_N

           1    02-43-275    24.4000      5
           2    02-57-161    24.5000      4
           3    04-31-143    25.6667      6
           4    04-36-007    25.3333      3
           5    04-37-206    25.3333      3

      3. merge by substr(hgtAgeHgt,4,2)  (BUG doc seems to indicate it should work)

         proc report data=havOne out=havOneSrt nowd;
           col hgtAgeWgt bmi;
           define hgtAgeWgt/order order=formated format= $hgtAgeWgt.;
         run;quit;

         proc report data=havTwo out=havTwoSrt nowd;
           col hgtAgeWgt bmi;
           define hgtAgeWgt/order order=formated format= $hgtAgeWgt.;
         run;quit;

          data wantMrg;
            format hgtAgeWgt $hgtAgeWgt.;
            merge havOneSrt havTwoSrt;
            by groupformat hgtAgeWgt;
          run;

      4. datastep by processing (another bug)

         proc report data=havOne out=havOneSrt nowd;
           col hgtAgeWgt bmi;
           define hgtAgeWgt/order order=formated format= $hgtAgeWgt.;
         run;quit;

         data wantDst;
           format hgtAgeWgt $hgtAgeWgt.;
           set havOneSrt;
           by groupformat hgtAgeWgt;
           if last.hgtAgeWgt;
         run;quit;


       5. proc freq fails

          Proc freq data=havOne;
          format hgtAgeWgt $hgtAgeWgt.;
          tables hgtAgeWgt;
          run;quit;


       6. proc print

          proc print data=havOne;
          format hgtAgeWgt $hgtAgeWgt.;
          var hgtAgeWgt;
          var bmi;
          run;quit;

       7. proc corresp;

          proc corresp data=havTwo  observed;
          tables hgtAgeWgt, bmi;
          run;quit;

       8. proc transpose

          proc transpose data=havTwo out=havXpo;
          format hgtAgeWgt $hgtAgeWgt.;
          var bmi;
          id hgtAgeWgt;
          run;quit;


    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;


    data havOne;
      retain hgtAgeWgt bmi;
      call streaminit(1234);
      do rec=1 to 100;
         hgtAgeWgt=catx('-',
              put(int(84*rand('uniform')),z2.)
             ,put(int(30*rand('uniform'))+30,z2.)
             ,put(int(300*rand('uniform')),z3.)
         );
         bmi=int(10*rand('uniform')) + 20;
         output;
         drop rec;
      end;
    run;quit;

    data havTwo;
      retain hgtAgeWgt bmi;
      call streaminit(1234);
      do rec=1 to 10;
         hgtAgeWgt=catx('-',
              put(int(84*rand('uniform')),z2.)
             ,put(int(30*rand('uniform'))+30,z2.)
             ,put(int(300*rand('uniform')),z3.)
         );
         bmi=int(10*rand('uniform')) + 20;
         output;
         drop rec;
      end;
    run;quit;



