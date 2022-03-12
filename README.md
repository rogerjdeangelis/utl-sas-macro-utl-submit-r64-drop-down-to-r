# utl-sas-macro-utl-submit-r64-drop-down-to-r
SAS macro utl_submit_r64 drop down to r 
    %let pgm=utl-sas-macro-utl-submit-r64-drop-down-to-r;

      SAS macro utl_submit_r64 drop down to r

      You can even execute R inside a datastep using DOSUBL at data execution time

      In this presentation I will demonstate the macro interface to R.

       Agenda

           1. Pass macro variable sex from SAS to R which contains M to Males subset
           2. R will read the sashelp.class table
           3. R will compute the mean weight for male students by age
           4. R will compute the total number of students in the class
           5. R will create a macro variable NUMBER_STUDENTS and pass this back to SAS
           6. R will create a transport file with mean male weights by age.
           7. The label portion of the V5 transport will have the long R variable names.
           8. SAS will rename the hashed short 8 character names to the long names stored in the V5
              transport varible labels..

    Macros on end and in github

    https://tinyurl.com/58pp9nz6
    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories

    This presentation is at

      https://tinyurl.com/3d2bbfbr
      https://github.com/rogerjdeangelis/utl-sas-macro-utl-submit-r64-drop-down-to-r

      and on Youtube

      https://www.youtube.com/channel/UCUzsiGhcv3OFovLJTazTc2w

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options  validvarname=upcase;

    libname sd1 "d:/sd1";

    data sd1.class;
      set sashelp.class;
    run;quit;

    /**************************************************************************************/
    /*                                                                                    */
    /* Up to 40 obs SD1.CLASS total obs=19 11MAR2022:12:00:44                             */
    /*                                                                                    */
    /* Obs    NAME       SEX    AGE    HEIGHT    WEIGHT                                   */
    /*                                                                                    */
    /*   1    Alfred      M      14     69.0      112.5                                   */
    /*   2    Alice       F      13     56.5       84.0                                   */
    /*   3    Barbara     F      13     65.3       98.0                                   */
    /*   4    Carol       F      14     62.8      102.5                                   */
    /*   5    Henry       M      14     63.5      102.5                                   */
    /*   6    James       M      12     57.3       83.0                                   */
    /*   7    Jane        F      12     59.8       84.5                                   */
    /*   8    Janet       F      15     62.5      112.5                                   */
    /*   9    Jeffrey     M      13     62.5       84.0                                   */
    /*  10    John        M      12     59.0       99.5                                   */
    /*  11    Joyce       F      11     51.3       50.5                                   */
    /*  12    Judy        F      14     64.3       90.0                                   */
    /*  13    Louise      F      12     56.3       77.0                                   */
    /*  14    Mary        F      15     66.5      112.0                                   */
    /*  15    Philip      M      16     72.0      150.0                                   */
    /*  16    Robert      M      12     64.8      128.0                                   */
    /*  17    Ronald      M      15     67.0      133.0                                   */
    /*  18    Thomas      M      11     57.5       85.0                                   */
    /*  19    William     M      15     66.5      112.0                                   */
    /*                                                                                    */
    /**************************************************************************************/

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    /**************************************************************************************/
    /*                                                                                    */
    /* NOTE: WE HAVE THE LONG VARIABLE NAMES USED IN THE R SCRIPT                         */
    /*                                                                                    */
    /*                                                                                    */
    /* Up to 40 obs WORK.MALES total obs=6 12MAR2022:06:52:48                             */
    /*                                                                                    */
    /*        MALE_     AVERAGE_    AGE_                                                  */
    /* Obs    GENDER       AGE      COUNT                                                 */
    /*                                                                                    */
    /*  1       M          11         1                                                   */
    /*  2       M          12         3                                                   */
    /*  3       M          13         1                                                   */
    /*  4       M          14         2                                                   */
    /*  5       M          15         2                                                   */
    /*  6       M          16         1                                                   */
    /*                                                                                    */
    /*  AND WE GET THE MACRO VARIABLE WITH NUMBER OF STUDENTS FROM R                      */
    /*                                                                                    */
    /*  %put STUDENTS IN CLASS TABLE &=NUMBER_STUDENTS;                                   */
    /*                                                                                    */
    /*  STUDENTS IN CLASS TABLE NUMBER_STUDENTS=19                                        */
    /*                                                                                    */
    /**************************************************************************************/
    /*                                                                                    */
    /* WHAT THE SHORT NAMES LOOK LIKE BEFORE WE RENAME                                    */
    /* THE LABEL CONTAINS THE LONG NAMESR NAMES                                           */
    /*                                                                                    */
    /* Variables in Creation Order                                                        */
    /*                                                                                    */
    /*  #    Variable    Type    Len    Label                                             */
    /*                                                                                    */
    /*  1    MALE_GEN    Char      1    MALE_GENDER                                       */
    /*  2    AVERAGE_    Num       8    AVERAGE_AGE                                       */
    /*  3    AGE_COUN    Char      1    AGE_COUNT                                         */
    /*                                                                                    */
    /**************************************************************************************/

    /*
     _ __  _ __ ___   ___ ___  ___ ___
    | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    */

    /* You need to clear the enviroment when doing development */

    %symdel number_students sex / nowarn;

    %utlfkil(d:/xpt/maleAges.xpt);

    proc datasets lib=work nodetails nolist mt=data mt=view;
     delete maleAges males;
    run;quit;

    /* we will pass this to R */
    %let sex=M;

    /* NOTE TSLIT WILL ADD SINGLE QUTES AROUND ALL THE R SCRIPT. ALLOW FOR MACRO RESOLUTION
       AND ADD EXTRA SINGLE QUATES AS NEEDED. IN THIS CASE IT WILL CHANGE '&sex' to ''&sex'' */;

    %utl_submit_r64(%tslit(
    library(sqldf);
    library(haven);
    library(SASxport);
    library(Hmisc);
    class<-read_sas("d:/sd1/class.sas7bdat");
    maleAges<-sqldf("
           select
               sex        as MALE_GENDER
              ,avg(age)   as AVERAGE_AGE
              ,count(age) as AGE_COUNT
           from
              class
           where
              sex='&sex'
           group
              by age
           ");
    maleAges;
    for (i in seq_along(maleAges)) {
              label(maleAges[,i])<- colnames(maleAges)[i];
           };
    NUMBER_STUDENTS<-nrow(class);
    writeClipboard(as.character(paste(NUMBER_STUDENTS, collapse = " ")));
    write.xport(maleAges,file="d:/xpt/maleAges.xpt");
    ),return=NUMBER_STUDENTS);

    %put STUDENTS IN SASHELP.CLASS &=NUMBER_STUDENTS;

    * rename variables in the transport file using names in the label;

    libname xpt xport "d:/xpt/maleAges.xpt";

    data males; /* you cannot use maleAges as the output table becuse there is a view called male Ages */

           %utl_rens(xpt.maleAges); /* creates view maleAges */

           set maleAges;

    run;quit;

    * WHAT THE SHORT NAMES IN THE V5 TRANSPORT LOOK LIKE;

    proc print data=xpt.maleAges;
    run;quit;


    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|
     _ __ ___   __ _  ___ _ __ ___  ___
    | `_ ` _ \ / _` |/ __| `__/ _ \/ __|
    | | | | | | (_| | (__| | | (_) \__ \
    |_| |_| |_|\__,_|\___|_|  \___/|___/

    */

    %macro utlfkil
        (
        utlfkil
        ) / des="delete an external file";


        /*-------------------------------------------------*\
        |                                                   |
        |  Delete an external file                          |
        |   From SAS macro guide                            |
        |  Sample invocations                               |
        |                                                   |
        |  WIN95                                            |
        |  %utlfkil(c:\dat\utlfkil.sas);                    |
        |                                                   |
        |                                                   |
        |  Solaris 2.5                                      |
        |  %utlfkil(/home/deangel/delete.dat);              |
        |                                                   |
        |                                                   |
        |  Roger DeAngelis                                  |
        |                                                   |
        \*-------------------------------------------------*/

        %local urc;

        /*-------------------------------------------------*\
        | Open file   -- assign file reference              |
        \*-------------------------------------------------*/

        %let urc = %sysfunc(filename(fname,%quote(&utlfkil)));

        /*-------------------------------------------------*\
        | Delete file if it exits                           |
        \*-------------------------------------------------*/

        %if &urc = 0 and %sysfunc(fexist(&fname)) %then %do;
            %let urc = %sysfunc(fdelete(&fname));
            %put xxxxxx &fname deleted xxxxxx;
        %end;
        %else %do;
            %put xxxxxx &fname not found xxxxxx;
        %end;

        /*-------------------------------------------------*\
        | Close file  -- deassign file reference            |
        \*-------------------------------------------------*/

        %let urc = %sysfunc(filename(fname,""));

      run;

    %mend utlfkil;

    %macro utl_rens(dsn);

      if _n_=0 then do;
        rc=%sysfunc(dosubl('

            data __ren001;
               set &dsn(obs=1);
            run;quit;

            proc transpose data=__ren001 out=__ren002(drop=col1);
              var _all_;
            run;quit;

            proc sql;
              select
                catx(' ',_name_,"as",lbl) into :rens separated by ","
              from
                (
                 select
                    _name_
                   ,case
                        when (_label_ = ' ') then _name_
                        else _label_
                    end as lbl
                 from
                    __ren002
                )
           ;quit;

            proc sql;
               create
                   view %scan(&dsn,2,'.')  as
               select
                   &rens
               from
                   &dsn.
            ;quit;
        '));
        drop rc;
      end;

    %mend utl_rens;


    %macro utl_submit_r64(
          pgmx
         ,return=NO
         ,resolve=NO
         )/des="Semi colon separated set of R commands - drop down to R";

      %utlfkil(%sysfunc(pathname(work))/r_pgm.txt);

      /* clear clipboard */
      filename _clp clipbrd;
      data _null_;
        file _clp;
        put " ";
      run;quit;

      * write the program to a temporary file;
      filename r_pgm "%sysfunc(pathname(work))/r_pgm.txt" lrecl=32766 recfm=v;

      data _null_;
        length pgm $32756;
        file r_pgm;
        if substr(upcase("&resolve"),1,1)="Y" then do;
            pgm=resolve(&pgmx);
         end;
        else do;
            pgm=&pgmx;
         end;
        put pgm;
        putlog pgm;
      run;

      %let __loc=%sysfunc(pathname(r_pgm));

      * pipe file through R;
      filename rut pipe "D:\r412\R\R-4.1.2\bin\R.exe --vanilla --quiet --no-save < &__loc";
      data _null_;
        file print;
        infile rut recfm=v lrecl=32756;
        input;
        put _infile_;
        putlog _infile_;
      run;

      filename rut clear;
      filename r_pgm clear;

      * use the clipboard to create macro variable;
      %if %upcase(&return) ne NO %then %do;
        filename clp clipbrd ;
        data _null_;
         infile clp;
         input;
         putlog "macro variable &return = " _infile_;
         call symputx("&return.",_infile_,"G");
        run;quit;
      %end;

    %mend utl_submit_r64;

    /*                                               _
     _ __ ___   __ _  ___ _ __ ___     ___ _ __   __| |
    | `_ ` _ \ / _` |/ __| `__/ _ \   / _ \ `_ \ / _` |
    | | | | | | (_| | (__| | | (_) | |  __/ | | | (_| |
    |_| |_| |_|\__,_|\___|_|  \___/   \___|_| |_|\__,_|

    */
