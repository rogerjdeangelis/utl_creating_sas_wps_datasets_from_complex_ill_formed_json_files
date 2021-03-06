# utl_creating_sas_wps_datasets_from_complex_ill_formed_json_files
Creating SAS or WPS dataset from a complex ill formed json files. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.
    Creating SAS, WPS or R dataframes from a complex ill formed json files;

      see  for jso mapping(schema? - one line of R code)
      https://goo.gl/Xr12pP
      https://github.com/rogerjdeangelis/utl_creating_sas_wps_datasets_from_complex_ill_formed_json_files

      github project
      https://github.com/rogerjdeangelis/utl_creating_sas_wps_datasets_from_complex_ill_formed_json_files

      inspired by
      https://goo.gl/SsdXGj
      https://stackoverflow.com/questions/48272441/how-to-remove-invalid-characters-in-a-json-file-in-r

      Should be helpful with XML and HTML too;

      see the R create json mapping

      Two solutions  WPS PROC R or IML/R

         1. Pick out the sublist (dataframe or SAS dataset) you want.
            This is deeply nested json file.

              res<-list[["data"]][["children"]][[2]][["data"]];

              * note you need to print out the R map (list structure to identify what you want)

              list is the R automatic mapping of json file
                 Note the misture of named and unnamed sublists

                   list[["data"]]  named
                   [[2]]          Unnamed


         2.  Have SAS parse the complete map (list structure)
             Not perfect but might be good enough?

    INPUT
    ====
      see for full json file
      https://www.reddit.com/r/burstcoin/new.json?sort=rising?limit=100

        * very small sample of the json file

         {"kind": "Listing", "data":
         {"after": "t3_7rvlag",
            "dist": 25, "modhash": "",
            "whitelist_status": null,
            "children": [
                {"kind": "t3", "data":
                {"domain": "self.burstcoin",
                   "approved_at_utc": null,
                   "mod_reason_by": null,
                   "banned_by": null,
                   "num_reports": null,
                   "subreddit_id": "t5_331i6",
                   "thumbnail_width": null,
                   "subreddit": "burstcoin",
                   "selftext_html": null,
                   "selftext": "",
                   "likes":null,
                   "suggested_sort": null,


    PROCESS (working code)
    ======================

         1. Pick out the sublist (dataframe or SAS dataset) you want

            json <- getURL("https://www.reddit.com/r/burstcoin/new.json?sort=rising?limit=100");
            json <- fromJSON(toJSON(json));
            lst <- RJSONIO::fromJSON(json);

           * the sublist you want;
            res<-lst[["data"]][["children"]][[2]][["data"]];  * need to print the map first ie str(res);

           * create data frame;
            want<-as.data.frame(unlist(res, recursive = TRUE, use.names = TRUE));

           * rownames to column;
            want$names <- rownames(want);

           * create SAS/WPS dataset;
            import r=want data=wrk.wantwps;

         2. Parse the R mapping in SAS

            * capture writes the json map to a text file;

            %utl_submit_wps64('
            libname sd1 sas7bdat "d:/sd1";
            options set=R_HOME "C:/Program Files/R/R-3.3.2";
            libname wrk sas7bdat "%sysfunc(pathname(work))";
            libname hlp sas7bdat "C:\Program Files\SASHome\SASFoundation\9.4\core\sashelp";
            proc r;
            submit;
            source("C:/Program Files/R/R-3.3.2/etc/Rprofile.site", echo=T);
            library(RCurl);
            library(RJSONIO);
            json <- getURL("https://www.reddit.com/r/burstcoin/new.json?sort=rising?limit=100");
            json <- fromJSON(toJSON(json));
            lst <- RJSONIO::fromJSON(json);
            capture.output(str(lst), file ="d:/txt/utl_creating_sas_wps_datasets_from_complex_ill_formed_json_files.txt");
            endsubmit;
            run;quit;
            ');


            options ls=256;
            data want;
              retain grp 0;
              length  variable $32 type $4 value $200;
              infile "d:/txt/utl_creating_sas_wps_datasets_from_complex_ill_formed_json_files.txt";
              *infile "d:/txt/utl_creating_sas_wps_datasets_from_complex_ill_formed_json_files_1.txt";
              input;
              if substr(_infile_,14,1)="$" then do;
                grp=grp+1;
                do until (index(_infile_, "$ distinguished")>0);
                   input;
                   if substr(_infile_,14,1)="$" then do;
                       variable=substr(_infile_,16,23);
                       type=substr(_infile_,41,4);
                       if type="NULL" then value="NULL";
                       else value=left(substr(_infile_,45));
                       output;
                   end;
                end;
              end;
            run;quit;


    OUTPUT
    ======

     1. Pick out the sublist (dataframe or SAS dataset) you want

        WORK.WANTWPS total obs=44

         VARIABLE                   VALUE

         domain                     self.burstcoin
         subreddit_id               t5_331i6
         subreddit                  burstcoin
         selftext
         is_reddit_media_domain     FALSE
         link_flair_text            Mining
         id                         7sdxo1
         archived                   FALSE
         clicked                    FALSE
         author                     Werwolf512
         num_crossposts             0
         saved                      FALSE
         can_mod_post               FALSE
         is_crosspostable           FALSE
         pinned                     FALSE
         score                      1
         over_18                    FALSE
         hidden                     FALSE
         thumbnail                  self
         edited                     FALSE
         link_flair_css_class       Mining
         contest_mode               FALSE
         gilded                     0
         downs                      0
         brand_safe                 FALSE
         stickied                   FALSE
         can_gild                   FALSE
         name                       t3_7sdxo1
         spoiler                    FALSE
         permalink                  /r/burstcoin/comments/7sdxo1/have_4x...
         subreddit_type             public
         locked                     FALSE
         hide_score                 FALSE
         created                    1516737862
         url                        https://www.reddit.com/r/burstcoin/c...
         quarantine                 FALSE
         title                      Have 4x 8TB HDD's, is it possible to...
         created_utc                1516709062
         subreddit_name_prefixed    r/burstcoin
         ups                        1


      2.  Have SAS parse the complete map (list structure)

        Not perfect but may be good enough

       GRP    VARIABLE                            TYPE    VALUE

         1    approved_at_utc                     NULL    NULL
         1    mod_reason_by                       NULL    NULL
         1    banned_by                           NULL    NULL
       ...
         1    mod_note                            NULL    NULL
         1    is_video                            logi    FALSE
         1    distinguished                       NULL    NULL

       ...

         2    approved_at_utc                     NULL    NULL
         2    mod_reason_by                       NULL    NULL
         2    banned_by                           NULL    NULL

         2    mod_note                            NULL    NULL
         2    is_video                            logi    FALSE
         2    distinguished                       NULL    NULL

       ...

        25    approved_at_utc                     NULL    NULL
        25    mod_reason_by                       NULL    NULL
        25    banned_by                           NULL    NULL
       ...
        25    mod_note                            NULL    NULL
        25    is_video                            logi    FALSE
        25    distinguished                       NULL    NULL

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

       see for full json file
       https://www.reddit.com/r/burstcoin/new.json?sort=rising?limit=100

    *      _      _                _     _ _     _
     _ __ (_) ___| | __  ___ _   _| |__ | (_)___| |_
    | '_ \| |/ __| |/ / / __| | | | '_ \| | / __| __|
    | |_) | | (__|   <  \__ \ |_| | |_) | | \__ \ |_
    | .__/|_|\___|_|\_\ |___/\__,_|_.__/|_|_|___/\__|
    |_|
    ;

    %utl_submit_wps64('
    libname sd1 sas7bdat "d:/sd1";
    options set=R_HOME "C:/Program Files/R/R-3.3.2";
    libname wrk sas7bdat "%sysfunc(pathname(work))";
    libname hlp sas7bdat "C:\Program Files\SASHome\SASFoundation\9.4\core\sashelp";
    proc r;
    submit;
    source("C:/Program Files/R/R-3.3.2/etc/Rprofile.site", echo=T);
    library(RCurl);
    library(RJSONIO);
    json <- getURL("https://www.reddit.com/r/burstcoin/new.json?sort=rising?limit=100");
    json <- fromJSON(toJSON(json));
    lst <- RJSONIO::fromJSON(json);
    res<-lst$data$children[[2]][["data"]];
    res<-lst[["data"]][["children"]][[2]][["data"]];
    want<-as.data.frame(unlist(res, recursive = TRUE, use.names = TRUE));
    want$Variable <- rownames(want);
    colnames(want) <- c("Value","Value));
    want;
    endsubmit;
    import r=want data=wrk.wantwps;
    run;quit;
    ');

    * wps log;

    > library(RJSONIO)
    > json <- getURL("https://www.reddit.com/r/burstcoin/new.json?sort=rising?limit=100")
    > json <- fromJSON(toJSON(json))
    > lst <- RJSONIO::fromJSON(json)
    > res<-lst$data$children[[2]][["data"]]
    > res<-lst[["data"]][["children"]][[2]][["data"]]
    > want<-as.data.frame(unlist(res, recursive = TRUE, use.names = TRUE))
    > want$Variable <- rownames(want)
    > colnames(want) <- c("Value","Value))
    + want

    NOTE: Processing of R statements complete

    20        import r=want data=wrk.wantwps;
    NOTE: Creating data set 'WRK.wantwps' from R data frame 'want'
    NOTE: Column names modified during import of 'want'
    NOTE: Data set "WRK.wantwps" has 44 observation(s) and 2 variable(s)

    21        run;
    NOTE: Procedure r step took :
          real time : 1.812


    *
     _ __   __ _ _ __ ___  ___   _ __ ___   __ _ _ __
    | '_ \ / _` | '__/ __|/ _ \ | '_ ` _ \ / _` | '_ \
    | |_) | (_| | |  \__ \  __/ | | | | | | (_| | |_) |
    | .__/ \__,_|_|  |___/\___| |_| |_| |_|\__,_| .__/
    |_|                                         |_|
    ;


    %utl_submit_wps64('
    libname sd1 sas7bdat "d:/sd1";
    options set=R_HOME "C:/Program Files/R/R-3.3.2";
    libname wrk sas7bdat "%sysfunc(pathname(work))";
    libname hlp sas7bdat "C:\Program Files\SASHome\SASFoundation\9.4\core\sashelp";
    proc r;
    submit;
    source("C:/Program Files/R/R-3.3.2/etc/Rprofile.site", echo=T);
    library(RCurl);
    library(RJSONIO);
    json <- getURL("https://www.reddit.com/r/burstcoin/new.json?sort=rising?limit=100");
    json <- fromJSON(toJSON(json));
    lst <- RJSONIO::fromJSON(json);
    capture.output(str(lst), file ="d:/txt/utl_creating_sas_wps_datasets_from_complex_ill_formed_json_files.txt");
    endsubmit;
    run;quit;
    ');

    options ls=256;
    data want;
      retain grp 0;
      length  variable $32 type $4 value $200;
      infile "d:/txt/utl_creating_sas_wps_datasets_from_complex_ill_formed_json_files.txt";
      *infile "d:/txt/utl_creating_sas_wps_datasets_from_complex_ill_formed_json_files_1.txt";
      input;
      if substr(_infile_,14,1)="$" then do;
        grp=grp+1;
        do until (index(_infile_, "$ distinguished")>0);
           input;
           if substr(_infile_,14,1)="$" then do;
               variable=substr(_infile_,16,23);
               type=substr(_infile_,41,4);
               if type="NULL" then value="NULL";
               else value=left(substr(_infile_,45));
               output;
           end;
        end;
      end;
    run;quit;

