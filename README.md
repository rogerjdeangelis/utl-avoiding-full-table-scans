# utl-avoiding-full-table-scans
Avoiding full table scans

    Avoiding full table scans                                                                                         
                                                                                                                      
    github                                                                                                            
    https://github.com/rogerjdeangelis/utl-avoiding-full-table-scans                                                  
                                                                                                                      
    This does not create additional variables but I find it hard to believe down wind processing                      
    cannot take advantage formatted values instead of physical variables.                                             
                                                                                                                      
    The code below creates a report with the additiona variables and even gives you an option                         
    to create a physical table with the extra columns,                                                                
                                                                                                                      
                                                                                                                      
    SAS Forum                                                                                                         
    https://tinyurl.com/v7y9pa3                                                                                       
    https://communities.sas.com/t5/SAS-Procedures/Using-specific-column-B-value-based-on-column-A-value/m-p/604018    
                                                                                                                      
    *_                   _                                                                                            
    (_)_ __  _ __  _   _| |_                                                                                          
    | | '_ \| '_ \| | | | __|                                                                                         
    | | | | | |_) | |_| | |_                                                                                          
    |_|_| |_| .__/ \__,_|\__|                                                                                         
            |_|                                                                                                       
    ;                                                                                                                 
                                                                                                                      
    data have;                                                                                                        
    input ID year pct_2010 pct_2011 pct_2012;                                                                         
    cards4;                                                                                                           
    1 2010 24 25 27                                                                                                   
    1 2011 24 25 27                                                                                                   
    1 2012 24 25 27                                                                                                   
    2 2010  .  1  .                                                                                                   
    2 2011  .  1  .                                                                                                   
    2 2012  .  1  .                                                                                                   
    3 2010 97 54 79                                                                                                   
    3 2011 97 54 79                                                                                                   
    3 2012 97 54 79                                                                                                   
    ;;;;                                                                                                              
    ;run;quit;                                                                                                        
                                                                                                                      
                                                                                                                      
    WORK.HAVE total obs=9                                                                                             
                                                                                                                      
      ID    YEAR    PCT_2010    PCT_2011    PCT_2012                                                                  
                                                                                                                      
       1    2010       24          25          27                                                                     
       1    2011       24          25          27                                                                     
       1    2012       24          25          27                                                                     
       2    2010        .           1           .                                                                     
       2    2011        .           1           .                                                                     
       2    2012        .           1           .                                                                     
       3    2010       97          54          79                                                                     
       3    2011       97          54          79                                                                     
       3    2012       97          54          79                                                                     
                                                                                                                      
    *            _               _                                                                                    
      ___  _   _| |_ _ __  _   _| |_                                                                                  
     / _ \| | | | __| '_ \| | | | __|                                                                                 
    | (_) | |_| | |_| |_) | |_| | |_                                                                                  
     \___/ \__,_|\__| .__/ \__,_|\__|                                                                                 
                    |_|                                                                                               
    ;                                                                                                                 
                                                                                                                      
    Proc report listing                                                                                               
                                                                       | Apply this map to PCT_2010-12                
                                                                       |                                              
      ID    YEAR    PCT_2010    PCT_2011    PCT_2012    V1    V2    V3 |                                              
                                                                       |                                              
       1    2010       24          25          27       1     2     2  | Cretae V1, V2 and V3 using this map          
       1    2011       24          25          27       1     2     2  |                                              
       1    2012       24          25          27       1     2     2  | 0   - < 25   = 1                             
       2    2010        .           1           .       4     1     4  | 25 -   75    = 2                             
       2    2011        .           1           .       4     1     4  | 75< -  high  = 3                             
       2    2012        .           1           .       4     1     4  | other        = 4                             
       3    2010       97          54          79       3     2     3  |                                              
       3    2011       97          54          79       3     2     3  |                                              
       3    2012       97          54          79       3     2     3  |                                              
                                                                                                                      
                                                                                                                      
    or if you really want a physical table with the extra columns (report option no additional code)                  
                                                                                                                      
    WORK.WANT total obs=9                                                                                             
                                                                                                                      
     ID    YEAR    PCT_2010    PCT_2011    PCT_2012    V1    V2    V3    _BREAK_                                      
                                                                                                                      
      1    2010       24          25          27       1     2     2                                                  
      1    2011       24          25          27       1     2     2                                                  
      1    2012       24          25          27       1     2     2                                                  
      2    2010        .           1           .       4     1     4                                                  
      2    2011        .           1           .       4     1     4                                                  
      2    2012        .           1           .       4     1     4                                                  
      3    2010       97          54          79       3     2     3                                                  
      3    2011       97          54          79       3     2     3                                                  
      3    2012       97          54          79       3     2     3                                                  
                                                                                                                      
                                                                                                                      
    *                                                                                                                 
     _ __  _ __ ___   ___ ___  ___ ___                                                                                
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|                                                                               
    | |_) | | | (_) | (_|  __/\__ \__ \                                                                               
    | .__/|_|  \___/ \___\___||___/___/                                                                               
    |_|                                                                                                               
    ;                                                                                                                 
    proc format;                                                                                                      
      value val2lvl                                                                                                   
      0   - < 25   = 1                                                                                                
      25 -   75    = 2                                                                                                
      75< -  high  = 3                                                                                                
      other        = 4                                                                                                
    ;run;quit;                                                                                                        
                                                                                                                      
                                                                                                                      
    proc report data=have nowd missing out=want threads;                                                              
     cols ID year pct_2010 pct_2011 pct_2012  pct_2010=v1 pct_2011=v2 pct_2012=v3;                                    
     format v1 v2 v3 val2lvl.;                                                                                        
     define v1 / display "v1"  f=val2lvl.;                                                                            
     define v2 / display "v2"  f=val2lvl.;                                                                            
     define v3 / display "v3"  f=val2lvl.;                                                                            
    run;quit;                                                                                                         
                                                                                                                      
                                                                                                                      
