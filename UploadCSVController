global with sharing class uploadCSVcontroller {

//    public Blob contentFile { get; set; }
//    public String nameFile { get; set; } 
//    public Integer rowCount { get; set; }
//    public Integer colCount { get; set; }
//    public String printResult {get;set;}
//    public List<List<String>> outputFile {get;set;}
	  public List<String> possibleOS2 = new List<String>();
	  public void deleteMyRecords(){
		  List<Joyent_Usage_Record__c> usgRecList = 
		  [SELECT ID FROM Joyent_Usage_Record__c limit 50000] ;
		   	  if(usgRecList.size() > 0){
				  delete usgRecList ; 
			  }				
	  }

public static void convertCSV(Blob contentFile){	 
    	
        List<List<String>> parsedCSV = new List<List<String>>();
        List<String> possibleOS2 = new List<String>();
        
        Date now = System.today();
        system.debug('System Date is ' + now);
        String tempNow = Zuora.zUtil.getZuoraDate(now);
        system.debug('Zuora system date = ' + tempNow);

        Integer rowCount = 0;
        Integer colCount = 0;
        
        
        if (contentFile != null){
        	
            String fileString = contentFile.toString(); 
            system.debug('This is the file content as a String: ' + fileString);
            
            parsedCSV = parseCSV(fileString, false);
            system.debug('This is the file content parsed: ' + parsedCSV);
            
            rowCount = parsedCSV.size();            
            system.debug('rowCount: ' + rowCount);  
            
            //List<List<String>> outputCSV = new String[rowCount];
            //List<String> outputCSV = new String[]{new List<String>(), new List<String>()};
            
            //create 2D List for List of rows in output CSV
            List<List<String>> outputCSV = new List<List<String>>();
            
            //create 1D list for each row 
            //List<String> tempList = new List<String>();
            List<String> tempList = new String[13];
            	
            List<Joyent_Usage_Record__c> usageRecs = new List<Joyent_Usage_Record__c>(); 
            
            
            //create record on Joyent_Usage_Record__c custom object in SF for each row in the parsed CSV file
            
            
            List<String> joyentPackages = new List<String>(); 
            	for(List<String> row : parsedCSV){
            		system.debug('creating usage rec for row ' + row);
            		String killNewLines = row[0].replace('\r','').replace('\n','').replace(' ','');
        			system.debug('killNewLines = ' + killNewLines);
            		Joyent_Usage_Record__c jur = new Joyent_Usage_Record__c();
            		jur.JOYENT_ACTION__c = row[0];
            		jur.MACHINE_ID__c = row[1];
            		jur.ACCOUNT_ID__c = row[2];
            		jur.DATE_TIME__c = row[3];
            		
//            		Boolean testRowMachine = row[0].startsWith('machine_create');
//            		system.debug('testRowMachine = ' + testRowMachine);
//            		Boolean testRowHvm = row[0].startsWith('hvm_destroy');
//            		system.debug('testRowHvm = ' + testRowHvm);
            		
            		String tempEscapeInsert = row[0].escapeCsv();
            		system.debug('tempEscapeInsert = ' + tempEscapeInsert);
            		
            		system.debug('row[0] before insert = ' + row[0]);
            		
            		if (killNewLines == 'machine_create'){
            			system.debug('entered if (row[0] == machine_create) condition');
            			List<String> joyentSize = row[4].split('v');
            			jur.JOYENT_SIZE__c = joyentSize[0];
            			//jur.JOYENT_SIZE__c = row[4];
            			system.debug('jur.JOYENT_SIZE__c = ' + jur.JOYENT_SIZE__c);
            			system.debug('current row.size() = ' + row.size());
            			//if(row.size()<7){
            			//	system.debug('row.size()<7');
            				List<String> joyentOS = row[5].split('-');
            				system.debug('joyentOS String list = ' + joyentOS);
            				jur.JOYENT_OS__c = joyentOS[0];
            			//}
            			system.debug('jur.JOYENT_OS__c = ' + jur.JOYENT_OS__c);
            			if(row.size()==7){
            				jur.JOYENT_OS2__c = row[6];
            				possibleOS2.add(row[6]);
            				system.debug('possibleOS2 = ' + possibleOS2);
            			}
            			
            		}else if(killNewLines =='hvm_destroy'){
            			system.debug('current row.size() = ' + row.size());
            			system.debug('entered if (row[0] == hvm_destroy) condition');
            			jur.JOYENT_SIZE__c = null;
            			jur.JOYENT_OS__c = null;
            			jur.JOYENT_OS2__c = null;
            		}else if(killNewLines == 'zone_resize'){
            			system.debug('current row.size() = ' + row.size());
            			system.debug('entered if (row[0] == zone_resize) condition ');
            			List<String> joyentSize = row[4].split('v');
            			jur.JOYENT_SIZE__c = joyentSize[0];
            			//List<String> joyentOS = row[5].split('-');
            			jur.JOYENT_OS__c = row[5];//joyentOS[1];
            			jur.JOYENT_OS2__c = row[6];
            		}else{
            			system.debug('current row.size() = ' + row.size());
            			system.debug('entered if (row[0] == zone_destroy) condition ');
            			jur.JOYENT_SIZE__c = null;
            			jur.JOYENT_OS__c = null;
            			jur.JOYENT_OS2__c = null;
            		}
            		
            		usageRecs.add(jur);
            	}
            	
            	system.debug('right before insert usageRecs');
            	 //resultList = new List<Database.SaveResult>();
            	
            	try{
            		List<Database.SaveResult> resultList = database.insert(usageRecs, true);
            		system.debug('resultList = ' + resultList); 
            		List<ID> recordId = new List<ID>();
            		// Iterate through each returned result
            		for( Database.SaveResult sr : resultList){
            			
            			String joyentAction = '';
            			// Insert was successful, get the ID of the record that was processed
            			if(sr.isSuccess()){
            				recordId.add(sr.getId());
            				system.debug('recordId = ' + recordId);
            			}
            			else{
            				// Operation failed, so get all errors                
        					for(Database.Error err : sr.getErrors()) {
        						System.debug('The following error has occurred.');                    
           						System.debug(err.getStatusCode() + ': ' + err.getMessage());
           						System.debug('Account fields that affected this error: ' + err.getFields());
       						}
            			}
            		}
            		system.debug('Pass Ids Method Call');
            		passIds(recordId);
            		
            		
            	} catch(DmlException e){
            		system.debug('ERROR: ' + e );
            		system.debug('File contents were not inserted into the Salesforce custom object Joyent_Usage_Record.');
            	}  	
        }else{
        	//Input file content is null or empty. Nothing to parse - write to error log
        	String emptyError = 'File is null or empty';
        	system.debug('ERROR: ' + emptyError);
   	    }
   	    
   	    system.debug('========================PARSING IS COMPLETE========================');
   	    	
}
//==============================================================================================================================
//===========================================================BEGIN PARSECSV METHOD===================================================================
    public static List<List<String>> parseCSV(String contents,Boolean skipHeaders) {
        
        List<List<String>> allFields = new List<List<String>>();
    
        // replace instances where a double quote begins a field containing a comma
        // in this case you get a double quote followed by a doubled double quote
        // do this for beginning and end of a field
        
        contents = contents.replaceAll(',"""',',"DBLQT').replaceall('""",','DBLQT",');
        
        // now replace all remaining double quotes - we do this so that we can reconstruct
        // fields with commas inside assuming they begin and end with a double quote
        
        contents = contents.replaceAll('""','DBLQT');
        
        // we are not attempting to handle fields with a newline inside of them
        // so, split on newline to get the spreadsheet rows
        
        List<String> lines = new List<String>();
        
        try {
            lines = contents.split('\n'); //correction: this only accomodates windows files
            //lines = contents.split('\r'); // using carriage return accomodates windows, unix, and mac files
            //http://www.maxi-pedia.com/Line+termination+line+feed+versus+carriage+return+0d0a
        } catch (System.ListException e) {
            System.debug('Limits exceeded?' + e.getMessage());
        }
        
        Integer num = 0;
        
        for(String line: lines) {
        	
            // check for blank CSV lines (only commas)
      
            if (line.replaceAll(',','').trim().length() == 0) break;
            
	            List<String> fields = line.split(',');  
	            List<String> cleanFields = new List<String>();
	            
	            String compositeField;
	            Boolean makeCompositeField = false;
	            
	            for(String field: fields) {
	                if (field.startsWith('"') && field.endsWith('"')) {
	                    cleanFields.add(field.replaceAll('DBLQT','"'));
	                } else if (field.startsWith('"')) {
	                    makeCompositeField = true;
	                    compositeField = field;
	                } else if (field.endsWith('"')) {
	                    compositeField += ',' + field;
	                    cleanFields.add(compositeField.replaceAll('DBLQT','"'));
	                    makeCompositeField = false;
	                } else if (makeCompositeField) {
	                    compositeField +=  ',' + field;
	                } else {
	                    cleanFields.add(field.replaceAll('DBLQT','"'));
	                }
            	}
            
            allFields.add(cleanFields);
        }
        
        if (skipHeaders) allFields.remove(0);
        return allFields;       
        
        //Download Method to download Table Data in Excel Sheet

    }         	
//==============================================================================================================================
//==============================================================BEGIN PARSTDATETIME METHOD================================================================            		
		public static Datetime parseDateTime(String parseString){
		//split the string by a specified delimeter		
		//EXAMPLE parseString: 2013-08-03T13:40:09+00:00 
		//Desired output date format = YYYY-MM-DDThh:mm:ss+hh:mm
		
		String year = parseString.substring(0, 4);
		system.debug('year = ' + year);
		String month = parseString.substring(5, 7);
		system.debug('month = ' + month);
		String day = parseString.substring(8, 10);
		system.debug('day = ' + day);
		
		String hour = parseString.substring(11,13);
		system.debug('hour = ' + hour);
		String minute = parseString.substring(14,16);
		system.debug('minute = ' + minute);
		String second = parseString.substring(17,19);
		system.debug('second = ' + second);
		
		//optional time zone offset 
		//String timezoneOffsethour = parseString.substring(20,22);
		//system.debug('timezoneOffsethour = ' + timezoneOffsethour);
		
		//String timezoneOffsetminute = parseString.substring(23,25);
		//system.debug('timezoneOffsetminute = ' + timezoneOffsetminute);
		
		String stringDate = year + '-' + month + '-' + day + ' ' + hour + ':' + minute + ':' + second; //+ '+' + timezoneOffsethour + ':' + timezoneOffsetminute;
		system.debug('stringDate format = ' + stringDate);
		
		Datetime resultDate = Datetime.valueOf(stringDate);
		system.debug('resultDate = ' + resultDate);
		
		return resultDate;
				
	}
//==============================================================================================================================
//==============================================================BEGIN GETQUANTITY METHOD================================================================	
	public static Double getQuantity(Datetime createdDateTime, Datetime destroyedDateTime, String machine_id ){
		
		Double quantity = 0.00000000000000000000;
				
        Double dt1 = createdDateTime.getTime() / 1000.0000000;  //getTime gives us milliseconds, so divide by 1000 for seconds  
        system.debug('createdDateTime = ' + dt1);
        
  		Double dt2 = destroyedDateTime.getTime() / 1000.0000000;
  		system.debug('destroyDateTime = ' + dt2);
  		
  		quantity = dt2 - dt1;
  		system.debug('quantity (in seconds) is ' + quantity);
  		
  		quantity = quantity/3600.0000000;
       	system.debug('quantity (in hours) is ' + quantity);
		//Need to write code that takes the Offset into account
		
	return quantity;
	
	}
//==============================================================================================================================	

//==============================================================BEGIN @FUTURE METHOD================================================================ 
 	@future(callout=true)		
	public static void passIds(List<ID> rId){
            	
            	system.debug('@future method begin>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>');
            	system.debug('lengthIdList = ' + rId.size());
            	
            	
            	List<List<Joyent_Usage_Record__c>> parsedCsvRow = new List<List<Joyent_Usage_Record__c>>();
            	List<List<String>> outputFile = new List<List<String>>();
            	
            		List<Joyent_Usage_Record__c> usageRecords =
	            	[SELECT 
	            	JOYENT_ACTION__c,
		            MACHINE_ID__c,
		            ACCOUNT_ID__c,  
		            DATE_TIME__c,
		            JOYENT_SIZE__c,
		            JOYENT_OS__c,
		            JOYENT_OS2__c
	            	FROM Joyent_Usage_Record__c
	            	WHERE ID IN: rId ];
	            	
            		system.debug('usageRecords = '+ usageRecords);
            		system.debug('usageREcords[0].JOYENT_ACTION__c = ' + usageREcords[0].JOYENT_ACTION__c );
            	           	
            	//EXAMPLE ROW: (machine_create, 7fc506cd-280f-4516-9e48-63b84c4891ba, 895e041e-e213-4963-b5f2-a2dc0bb11085, 2013-08-03T13:40:09+00:00, Small_80GB, xampp-linux-1.8.2)
            	
	            String endDateTime = '';
	            String subId = '';
	            String chargeId = '';
	            String description = '';
	            String dataCenter = '';
	            //List<LB_Lookup_Table__c> listoftables = new List<LB_Lookup_Table__c>();
            	
            	system.debug('before for loop through rows in parsedCSV');
            	Integer forLoopCounter =1;
            	Integer ifStatementCounter =1;
            	
            	for(Joyent_Usage_Record__c row : usageRecords){
            		
            		List<LB_Lookup_Table__c> listoftables = new List<LB_Lookup_Table__c>();
            		
            		system.debug('row is: ' + row);
            		
            		system.debug('========================================For loop iteration number========================================> ' + forLoopCounter);
            		//
            		Double quantity = 0.0000;
            		String ZuoraUOM;
            		
            		List<String> outputCSVrow = new List<String>();
            		
        			//String removeNewLines = row[0].replace('\r','').replace('\n','').replace(' ','');
        			//system.debug('removeNewLines = ' + removeNewLines);
        			
        			//String tempEscape = row[0].escapeCsv();
            		//system.debug('tempEscape = ' + tempEscape);
	            		
	            		//if(removeNewLines.startsWith('machine'))
	            		if(row.JOYENT_ACTION__c == 'machine_create' ){
	            			
	            			system.debug('========================================If Statement iteration number========================================> ' + ifStatementCounter);
	            			system.debug('entered machine_create if statement, before query');
	            			
	            			//Query the J Usage Object for an HVM destroy Date with the same machine ID
	            			List<Joyent_Usage_Record__c> createdUsageRec = 
	            			[Select 
	            			JOYENT_ACTION__c,
	            			MACHINE_ID__c,
	            			DATE_TIME__c
							FROM Joyent_Usage_Record__c 
							WHERE MACHINE_ID__c =: row.MACHINE_ID__c
							AND JOYENT_ACTION__c = 'hvm_destroy'];
							
							system.debug('createdUsageRec = ' + createdUsageRec);
							system.debug('createdUsageRec.size() = ' + createdUsageRec.size());
							
							//Query the J Usage Object for a Zone_Destroy date with the same machine ID
							List<Joyent_Usage_Record__c> createdUsageRec_ZONE = 
	            			[Select 
	            			JOYENT_ACTION__c,
	            			MACHINE_ID__c,
	            			DATE_TIME__c
							FROM Joyent_Usage_Record__c 
							WHERE MACHINE_ID__c =: row.MACHINE_ID__c 
							AND JOYENT_ACTION__c = 'zone_destroy'];
							
							system.debug('createdUsageRec_ZONE = ' + createdUsageRec_ZONE);
							system.debug('createdUsageRec_ZONE.size() = ' + createdUsageRec_ZONE.size());
							
							if (createdUsageRec.size() > 0 && createdUsageRec_ZONE.size()==0){
								
								system.debug('HVM DESTROY DATE EXISTS. ZONE DESTROY DOES NOT FOR MACHINE ID ' + createdUsageRec[0].MACHINE_ID__c);
								system.debug('row = ' + row);
								system.debug('===================HVM_DESTROY DATE EXISTS FOR MACHINE ID: ' + createdUsageRec[0].MACHINE_ID__c + ' AND IT = ' + createdUsageRec[0].DATE_TIME__c);
								
								String createdDateTimeS = row.DATE_TIME__c;
								//String createdDateTimeS = '2013-08-03T13:40:09+00:00';
	            				system.debug('createdUsageRec (createdDateTimeS)= ' + createdDateTimeS);
	            			
	            				String destroyedDateTimeS = createdUsageRec[0].DATE_TIME__c;
	            				system.debug('destroyedDateTimeS = ' + createdUsageRec[0].DATE_TIME__c);
	            			
	            				//EXAMPLE DATE: 2013-08-03T13:40:09+00:00 
	            				//EX FORMAT:    YYYY-MM-DDThh:mm:ss+hh:mm            			
	            				Datetime createdDateTime = parseDateTime(createdDateTimeS);
	            				system.debug('createdDateTime = ' + createdDateTime);
	            				String createdDateTimeString = String.valueOf(createdDateTime);
	            			            			
	            				DateTime destroyedDateTime = parseDateTime(destroyedDateTimeS);
	            				system.debug('destroyedDateTime = ' + destroyedDateTime);
	            				String destroyedDateTimeString = String.valueOf(destroyedDateTime);
	            			
	            				String createdUsageRecString = createdUsageRec[0].MACHINE_ID__c;
	            				system.debug('createdUsageRec[0].MACHINE_ID__c (createdUsageRecString) = ' + createdUsageRec[0].MACHINE_ID__c);
	            				
	            				//calculate the quantity from the dates for the machineID specified
	            				Double unitsUsed = getQuantity(createdDateTime, destroyedDateTime, createdUsageRecString);
	            				quantity = unitsUsed;
	            				String qString = String.valueOf(quantity);
	            			
	            				//outputCSVrow[0] = ;
	            				//system('first line of output is ' + );
	            			
	            				//Always set the Usage.StartDate to the last day of the current month (so if today is July 7th, we want this to be 7/31/2013)
	            				//     			datetime myDateTime = datetime.now();
	            			
	            				system.debug('createdUsageRec[0].JOYENT_ACTION__c = ' + createdUsageRec[0].JOYENT_ACTION__c);
	            				system.debug('createdUsageRec[0].MACHINE_ID__c = ' + createdUsageRec[0].MACHINE_ID__c);
	            				system.debug('createdUsageRec[0].DATE_TIME__c = ' + createdUsageRec[0].DATE_TIME__c);
	            				
	            				//List <String> queryJoyentSize = new List<String>();
	            				//queryJoyentSize = row[4].split('v');
	            				
	            				//List<String> queryJoyentOS = new List<String>();            			
	            				//queryJoyentOS = row[5].split('-');    
	            				        			
	            				if (row.JOYENT_OS2__c == null){
	            					system.debug('Destroy date exists and row size is less than 7');
	            					listoftables = 
	            					[SELECT Id, Zuora_UOM__c 
	            					FROM LB_Lookup_Table__c 
	            					WHERE Joyent_Size__c =: row.JOYENT_SIZE__c
	            					AND Joyent_OS__c =: row.JOYENT_OS__c LIMIT 1];
	            				} else{
	            					system.debug('Destroy date exists and row size is greater than 7');
	            					listoftables = 
	            					[SELECT Id, Zuora_UOM__c 
	            					FROM LB_Lookup_Table__c 
	            					WHERE Joyent_Size__c =: row.JOYENT_SIZE__c
	            					//AND Joyent_OS__c =: row[5]  
	            					AND JOYENT_OS2__c =: 'SmartOS' LIMIT 1];
	            				}
	            			      
	            				if(listoftables.size()>0){
		            				ZuoraUOM = listoftables[0].Zuora_UOM__c;
		            				system.debug('ZuoraUOM = ' + ZuoraUOM);
	            				}
	            			
	            			           			
	            				system.debug('listoftables = ' + listoftables);
	            				
	            				Date obj = Date.today();
	            				system.debug(obj);
	            				
	            				Integer year = obj.year();
	            				system.debug('obj.year = ' + year);
	            				
	            				Integer month = obj.month();
	            				Integer numDays = date.daysInMonth(year, month);
	            				String parsedDate = '';
	            				
	            				system.debug('month = ' + month);
	            				
	            				if(month<10){
	            					parsedDate = '0' + month + '/' + numDays + '/' + year;
	            					system.debug('parsedDate = ' + parsedDate);
	            				}else{
	            					parsedDate = month + '/' + numDays + '/' + year;
	           						system.debug('parsedDate with month >= 10 = ' + parsedDate);
	           					}
	            					
	           					outputCSVrow.add(row.ACCOUNT_ID__c);
		            			outputCSVrow.add(ZuoraUOM);
		            			outputCSVrow.add(qString);
		            			outputCSVrow.add(parsedDate);
		            			outputCSVrow.add(endDateTime);
		            			outputCSVrow.add(subId);
		            			outputCSVrow.add(chargeId);
		            			outputCSVrow.add(description);
		            			outputCSVrow.add(dataCenter);
		            			outputCSVrow.add(createdDateTimeString);
		            			outputCSVrow.add(destroyedDateTimeString);			            			
		            			outputCSVrow.add(row.MACHINE_ID__c);
			            		outputCSVrow.add('1');
			            		system.debug('outputCSVrow = ' + outputCSVrow);
			            		
			           			outputFile.add(outputCSVrow);
			            			system.debug('outputFile = ' + outputFile);
			            			system.debug('outputFile size = ' + outputFile.size());
			            			
			            			if(outputFile.size() > 1){
			            				for (Integer n=0; n < outputFile.size(); n++){
			            					system.debug('outputFile[' + n + '] = ' + outputFile[n]);
			            				}
			            			}else if(outputFile.size() == 1){
			            				system.debug('outputFile[0] = ' + outputFile[0]);
			            			}
	            				
	            			
							}else if(createdUsageRec.size()== 0 && createdUsageRec_ZONE.size() > 0){
								system.debug('====================HVM_DESTROY DATE DOES NOT EXIST BUT ZONE DESTROY DOES====================');
								system.debug('row = ' + row);
								
								String createdDateTimeS = row.DATE_TIME__c;
								//String createdDateTimeS = '2013-08-03T13:40:09+00:00';
	            				system.debug('createdUsageRec_ZONE (createdDateTimeS)= ' + createdDateTimeS);
	            			
	            				String destroyedDateTimeS = createdUsageRec_ZONE[0].DATE_TIME__c;
	            				system.debug('destroyedDateTimeS = ' + createdUsageRec_ZONE[0].DATE_TIME__c);
	            			
	            				//EXAMPLE DATE: 2013-08-03T13:40:09+00:00 
	            				//EX FORMAT:    YYYY-MM-DDThh:mm:ss+hh:mm            			
	            				Datetime createdDateTime = parseDateTime(createdDateTimeS);
	            				system.debug('createdDateTime = ' + createdDateTime);
	            				String createdDateTimeString = String.valueOf(createdDateTime);
	            			            			
	            				DateTime destroyedDateTime = parseDateTime(destroyedDateTimeS);
	            				system.debug('destroyedDateTime = ' + destroyedDateTime);
	            				String destroyedDateTimeString = String.valueOf(destroyedDateTime);
	            			
	            				String createdUsageRecString = createdUsageRec_ZONE[0].MACHINE_ID__c;
	            				system.debug('createdUsageRec_ZONE[0].MACHINE_ID__c (createdUsageRecString) = ' + createdUsageRec_ZONE[0].MACHINE_ID__c);
	            				
	            				//calculate the quantity from the dates for the machineID specified
	            				Double unitsUsed = getQuantity(createdDateTime, destroyedDateTime, createdUsageRecString);
	            				quantity = unitsUsed;
	            				String qString = String.valueOf(quantity);
	            			
	            				//outputCSVrow[0] = ;
	            				//system('first line of output is ' + );
	            			
	            				//Always set the Usage.StartDate to the last day of the current month (so if today is July 7th, we want this to be 7/31/2013)
	            				//     			datetime myDateTime = datetime.now();
	            			
	            				system.debug('createdUsageRec_ZONE[0].JOYENT_ACTION__c = ' + createdUsageRec_ZONE[0].JOYENT_ACTION__c);
	            				system.debug('createdUsageRec_ZONE[0].MACHINE_ID__c = ' + createdUsageRec_ZONE[0].MACHINE_ID__c);
	            				system.debug('createdUsageRec_ZONE[0].DATE_TIME__c = ' + createdUsageRec_ZONE[0].DATE_TIME__c);
	            				
	            				//List <String> queryJoyentSize = new List<String>();
	            				//queryJoyentSize = row[4].split('v');
	            				
	            				//List<String> queryJoyentOS = new List<String>();            			
	            				//queryJoyentOS = row[5].split('-');    
	            				
	            				String[] possibleOS2 = new String[]{'SmartOS', 'riak', 'mongodb', 'mysql', 'hadoop'};
	            				if (row.JOYENT_OS2__c == null){
	            					system.debug('Destroy date exists and row size is less than 7');
	            					listoftables = 
	            					[SELECT Id, Zuora_UOM__c 
	            					FROM LB_Lookup_Table__c 
	            					WHERE Joyent_Size__c =: row.JOYENT_SIZE__c
	            					AND Joyent_OS__c =: row.JOYENT_OS__c LIMIT 1];
	            				} else{
	            					system.debug('Destroy date exists and row size is greater than 7');
	            					listoftables = 
	            					[SELECT Id, Zuora_UOM__c 
	            					FROM LB_Lookup_Table__c 
	            					WHERE Joyent_Size__c =: row.JOYENT_SIZE__c
	            					//AND Joyent_OS__c =: row[5] 
	            					//AND JOYENT_OS2__c =: 'SmartOS' LIMIT 1];
	            					AND JOYENT_OS2__c IN :possibleOS2 LIMIT 1];
	            				}
	            			      
	            				if(listoftables.size()>0){
		            				ZuoraUOM = listoftables[0].Zuora_UOM__c;
		            				system.debug('ZuoraUOM = ' + ZuoraUOM);
	            				}
	            			
	            			           			
	            				system.debug('listoftables = ' + listoftables);
	            				
	            				Date obj = Date.today();
	            				system.debug(obj);
	            				
	            				Integer year = obj.year();
	            				system.debug('obj.year = ' + year);
	            				
	            				Integer month = obj.month();
	            				Integer numDays = date.daysInMonth(year, month);
	            				String parsedDate = '';
	            				
	            				system.debug('month = ' + month);
	            				
	            				if(month<10){
	            					parsedDate = '0' + month + '/' + numDays + '/' + year;
	            					system.debug('parsedDate = ' + parsedDate);
	            				}else{
	            					parsedDate = month + '/' + numDays + '/' + year;
	           						system.debug('parsedDate with month >= 10 = ' + parsedDate);
	           					}
	            					
	           					outputCSVrow.add(row.ACCOUNT_ID__c);
		            			outputCSVrow.add(ZuoraUOM);
		            			outputCSVrow.add(qString);
		            			outputCSVrow.add(parsedDate);
		            			outputCSVrow.add(endDateTime);
		            			outputCSVrow.add(subId);
		            			outputCSVrow.add(chargeId);
		            			outputCSVrow.add(description);
		            			outputCSVrow.add(dataCenter);
		            			outputCSVrow.add(createdDateTimeString);
		            			outputCSVrow.add(destroyedDateTimeString);			            			
		            			outputCSVrow.add(row.MACHINE_ID__c);
			            		outputCSVrow.add('1');
			            		system.debug('outputCSVrow = ' + outputCSVrow);
			            		
			           			outputFile.add(outputCSVrow);
			            			system.debug('outputFile = ' + outputFile);
			            			system.debug('outputFile size = ' + outputFile.size());
			            			
			            			if(outputFile.size() > 1){
			            				for (Integer n=0; n < outputFile.size(); n++){
			            					system.debug('outputFile[' + n + '] = ' + outputFile[n]);
			            				}
			            			}else if(outputFile.size() == 1){
			            				system.debug('outputFile[0] = ' + outputFile[0]);
			            			}
								
							}else if (createdUsageRec.size()== 0 && createdUsageRec_ZONE.size()==0){
								system.debug('====================HVM_DESTROY DATE DOES NOT EXIST and neither does ZONE_DESTROY====================');
								system.debug('row = ' + row);
								String createdDateTimeS = row.DATE_TIME__c;
								//String createdDateTimeS = '2013-08-03T13:40:09+00:00';
	            				system.debug('createdUsageRec (createdDateTimeS)= ' + createdDateTimeS);
	            			
	            				Date currentDate = Date.today();
	            				Integer cyear = currentDate.year();
	            				system.debug('currentDate.year = ' + cyear);
	            				
	            				Integer cmonth = currentDate.month();
	            				
	            				Integer cday = currentDate.day();
	            				
	            				String currentDateString = '';
	            				
	            				if(cmonth<10){
	            					currentDateString =  cyear + '-' + '0' + cmonth + '-' + cday + ' ' + '00' + ':' + '00' + ':' + '00';
	            					system.debug('currentDateString = ' + currentDateString);
	            				}else{
	            					currentDateString = cyear + '-' + cmonth + '-' + cday + ' ' + '00' + ':' + '00' + ':' + '00';
	           						system.debug('currentDateString = ' + currentDateString);
	           					}
	           					
	           					
	            				String destroyedDateTimeS = currentDateString;
	            				//hard code end date to test
	            				destroyedDateTimeS = '2013-12-31 23:59:59';
	            				system.debug('destroyedDateTimeS = ' + Date.today());
	            			
	            				//EXAMPLE DATE: 2013-08-03T13:40:09+00:00 
	            				//EX FORMAT:    YYYY-MM-DDThh:mm:ss+hh:mm            			
	            				Datetime createdDateTime = parseDateTime(createdDateTimeS);
	            				system.debug('createdDateTime = ' + createdDateTime);
	            				String createdDateTimeString = String.valueOf(createdDateTime);
	            			            			
	            				DateTime destroyedDateTime = parseDateTime(destroyedDateTimeS);
	            				system.debug('destroyedDateTime = ' + destroyedDateTime);
	            				String destroyedDateTimeString = String.valueOf(destroyedDateTime);
	            			
	            				String createdUsageRecString = row.MACHINE_ID__c;
	            				system.debug('MACHINE_ID__c (createdUsageRecString) = ' + createdUsageRecString);
	            				
	            				//calculate the quantity from the dates for the machineID specified
	            				Double unitsUsed = getQuantity(createdDateTime, destroyedDateTime, createdUsageRecString);
	            				quantity = unitsUsed;
	            				String qString = String.valueOf(quantity);
	            			
	            				//outputCSVrow[0] = ;
	            				//system('first line of output is ' + );
	            			
	            				//Always set the Usage.StartDate to the last day of the current month (so if today is July 7th, we want this to be 7/31/2013)
	            				//     			datetime myDateTime = datetime.now();
	            			
	            				system.debug('JOYENT_ACTION__c = ' + row.JOYENT_ACTION__c);
	            				system.debug('MACHINE_ID__c = ' + row.MACHINE_ID__c);
	            				system.debug('DATE_TIME__c = ' + row.DATE_TIME__c);
	            				
	            				//List <String> queryJoyentSizeB = new List<String>();
	            				//queryJoyentSizeB = row[4].split('v');
	            				
	            				//List<String> queryJoyentOSB = new List<String>();            			
	            				//queryJoyentOSB = row[5].split('-');
	            				
	            				String[] possibleOS2 = new String[]{'SmartOS', 'riak', 'mongodb', 'mysql', 'hadoop'};
	            				if (row.JOYENT_OS2__c == null){
	            					system.debug('HVM Destroy date DOES NOT exist and row size is less than 7');
	            					listoftables = 
	            					[SELECT 
	            						Id, 
	            						Zuora_UOM__c 
	            					FROM 
	            						LB_Lookup_Table__c 
	            					WHERE 
	            						Joyent_Size__c =: row.JOYENT_SIZE__c
	            					AND 
	            						Joyent_OS__c =: row.JOYENT_OS__c LIMIT 1];
	            				} else{
	            					system.debug('HVM Destroy date DOES NOT exist and row size is greater than 7');
	            					listoftables = 
	            					[SELECT Id, Zuora_UOM__c 
	            					FROM LB_Lookup_Table__c 
	            					WHERE Joyent_Size__c =: row.JOYENT_SIZE__c
	            					//AND Joyent_OS__c =: row[5] 
	            					//AND JOYENT_OS2__c =: row.JOYENT_OS2__c LIMIT 1];
	            					AND JOYENT_OS2__c IN :possibleOS2 LIMIT 1];
	            				}  
	            			      
	            				if(listoftables.size()>0){
		            				ZuoraUOM = listoftables[0].Zuora_UOM__c;
		            				system.debug('ZuoraUOM = ' + ZuoraUOM);
	            				}
	            			
	            			           			
	            				system.debug('listoftables = ' + listoftables);
	            				
	            				Date obj = Date.today();
	            				system.debug(obj);
	            				
	            				Integer year = obj.year();
	            				system.debug('obj.year = ' + year);
	            				
	            				Integer month = obj.month();
	            				Integer numDays = date.daysInMonth(year, month);
	            				String parsedDate = '';
	            				
	            				if(month<10){
	            					parsedDate = '0' + month + '/' + numDays + '/' + year;
	            					system.debug('parsedDate = ' + parsedDate);
	            				}else{
	            					parsedDate = month + '/' + numDays + '/' + year;
	           						system.debug(parsedDate);
	           					}
	            				
	           					outputCSVrow.add(row.ACCOUNT_ID__c);
		            			outputCSVrow.add(ZuoraUOM);
		            			outputCSVrow.add(qString);
		            			outputCSVrow.add(parsedDate);
		            			outputCSVrow.add(endDateTime);
		            			outputCSVrow.add(subId);
		            			outputCSVrow.add(chargeId);
		            			outputCSVrow.add(description);
		            			outputCSVrow.add(dataCenter);
		            			outputCSVrow.add(createdDateTimeString);
		            			outputCSVrow.add(destroyedDateTimeString);			            			
		            			outputCSVrow.add(row.MACHINE_ID__c);
			            		outputCSVrow.add('1');
			            		system.debug('outputCSVrow = ' + outputCSVrow);
			            		system.debug('outputCSVrow[0] = ' + outputCSVrow[0]);
			            		system.debug('outputCSVrow[1] = ' + outputCSVrow[1]);
			            		system.debug('outputCSVrow[2] = ' + outputCSVrow[2]);
			            		system.debug('outputCSVrow[3] = ' + outputCSVrow[3]);
			            		system.debug('outputCSVrow[4] = ' + outputCSVrow[4]);
			            		system.debug('outputCSVrow[5] = ' + outputCSVrow[5]);
			            		system.debug('outputCSVrow[6] = ' + outputCSVrow[6]);
			            		system.debug('outputCSVrow[7] = ' + outputCSVrow[7]);
			            		system.debug('outputCSVrow[8] = ' + outputCSVrow[8]);
			            		system.debug('outputCSVrow[9] = ' + outputCSVrow[9]);
			            		system.debug('outputCSVrow[10] = ' + outputCSVrow[10]);
			            		system.debug('outputCSVrow[11] = ' + outputCSVrow[11]);
			            		system.debug('outputCSVrow[12] = ' + outputCSVrow[12]);
			            		
			           			outputFile.add(outputCSVrow);
			            			system.debug('outputFile = ' + outputFile);
			            			system.debug('outputFile size = ' + outputFile.size());
			            			
			            			if(outputFile.size() > 1){
			            				for (Integer n=0; n < outputFile.size(); n++){
			            					system.debug('outputFile[' + n + '] = ' + outputFile[n]);
			            				}
			            			}else if(outputFile.size() == 1){
			            				system.debug('outputFile[0] = ' + outputFile[0]);
			            			}
								
							}				

	            			
	            		ifStatementCounter++;
	            		
	            		//}else if(removeNewLines.startsWith('zone_resize')){
	            		}else if(row.JOYENT_ACTION__c == 'zone_resize'){
	            			system.debug('****************************************First Value of Row is a Zone Method****************************************');
	            			
	            			system.debug('entered zone_resize if statement, before query');
	            			
	            			List<Joyent_Usage_Record__c> createdUsageRecMachineCreate = 
	            			[Select 
	            			JOYENT_ACTION__c,
	            			MACHINE_ID__c,
	            			ACCOUNT_ID__c,
	            			DATE_TIME__c,
	            			JOYENT_SIZE__c,
	            			JOYENT_OS__c,
	            			JOYENT_OS2__c
							FROM Joyent_Usage_Record__c 
							WHERE MACHINE_ID__c =: row.MACHINE_ID__c 
							AND JOYENT_ACTION__c = 'machine_create'];
							
							system.debug('createdUsageRecMachineCreate = ' + createdUsageRecMachineCreate);
							system.debug('createdUsageRecMachineCreate.size() = ' + createdUsageRecMachineCreate.size());
							
							List<Joyent_Usage_Record__c> createdUsageRecZoneDestroy = 
	            			[Select 
	            			JOYENT_ACTION__c,
	            			MACHINE_ID__c,
	            			DATE_TIME__c
							FROM Joyent_Usage_Record__c 
							WHERE MACHINE_ID__c =: row.MACHINE_ID__c 
							AND JOYENT_ACTION__c = 'zone_destroy'];
							
							system.debug('createdUsageRecZoneDestroy = ' + createdUsageRecZoneDestroy);
							system.debug('createdUsageRecZoneDestroy.size() = ' + createdUsageRecZoneDestroy.size());
							
							List<Joyent_Usage_Record__c> createdUsageRecHVMDestroy = 
	            			[Select 
	            			JOYENT_ACTION__c,
	            			MACHINE_ID__c,
	            			DATE_TIME__c
							FROM Joyent_Usage_Record__c 
							WHERE MACHINE_ID__c =: row.MACHINE_ID__c 
							AND JOYENT_ACTION__c = 'hvm_destroy'];
							
							system.debug('createdUsageRecHVMDestroy = ' + createdUsageRecHVMDestroy);
							system.debug('createdUsageRecHVMDestroy.size() = ' + createdUsageRecHVMDestroy.size());
							
							
							//if machine_create date and zone destroy date exists, calculate usage between those dates
							if (createdUsageRecMachineCreate.size()>0 && createdUsageRecZoneDestroy.size()>0){
								system.debug('========================================Machine Create Record exists and so does Zone Destroy===================');
								system.debug('row = ' + row);
								system.debug('entered if (createdUsageRecMachineCreate.size()>0 && createdUsageRecZoneDestroy.size()>0) statement');
								
								
								//1 - Calculate usage quantity between machine_create date and zone_destroy date
								
								String machineCreateDate = createdUsageRecMachineCreate[0].DATE_TIME__c;
								system.debug('createdUsageRecMachineCreate[0].DATE_TIME__c = ' + createdUsageRecMachineCreate[0].DATE_TIME__c);
								String zoneDestroyDate = createdUsageRecZoneDestroy[0].DATE_TIME__c;
								system.debug('createdUsageRecZoneDestroy[0].DATE_TIME__c = ' + createdUsageRecZoneDestroy[0].DATE_TIME__c);
								
								Datetime createdMachineCreateDate = parseDateTime(machineCreateDate);
	            				String createdMachineDateTimeString = String.valueOf(createdMachineCreateDate);
	            				
	            				Datetime createdZoneDestroyDate = parseDateTime(zoneDestroyDate);
	            				String createdZoneDestroyDateTimeString = String.valueOf(createdZoneDestroyDate);
	            				
	            				String machineId = createdUsageRecMachineCreate[0].MACHINE_ID__c;
	            				Double hoursUsed = getQuantity(createdMachineCreateDate, createdZoneDestroyDate, machineId);
	            				quantity = hoursUsed;
	            				String quantityUsed = String.valueOf(quantity);
	            				system.debug('quantityUsed = ' + quantityUsed);
	            				
	            				//Query for UOM
	            				List <String> zoneResize_Size = new List<String>();
	            				zoneResize_Size = createdUsageRecMachineCreate[0].JOYENT_SIZE__c.split('v');  
	            				        			
	            				system.debug('zoneResize_Size[0] = ' + zoneResize_Size[0]);
	            				system.debug('createdUsageRecMachineCreate[0].JOYENT_OS2__c = ' + createdUsageRecMachineCreate[0].JOYENT_OS2__c);
	            				listoftables = 
	            				[SELECT Id, Zuora_UOM__c 
	            				FROM LB_Lookup_Table__c 
	            				WHERE Joyent_Size__c =: zoneResize_Size[0]
	            				//AND Joyent_OS__c =: row[5] 
	            				AND JOYENT_OS2__c =: createdUsageRecMachineCreate[0].JOYENT_OS2__c LIMIT 1];
	            				
	            				if(listoftables.size()>0){
		            				ZuoraUOM = listoftables[0].Zuora_UOM__c;
		            				system.debug('ZuoraUOM = ' + ZuoraUOM);
	            				}
	            			           			
	            				system.debug('listoftables = ' + listoftables);
	            			
	            				//Create row in output file to send to Zuora
	            				
	            				Date obj = Date.today();
	            				system.debug(obj);
	            				
	            				Integer year = obj.year();
	            				system.debug('obj.year = ' + year);
	            				
	            				Integer month = obj.month();
	            				Integer numDays = date.daysInMonth(year, month);
	            				String parsedDate = '';
	            				
	            				if(month<10){
	            					parsedDate = '0' + month + '/' + numDays + '/' + year;
	            					system.debug('parsedDate = ' + parsedDate);
	            				}else{
	            					parsedDate = month + '/' + numDays + '/' + year;
	           						system.debug(parsedDate);
	           					}
	            					
	           					outputCSVrow.add(createdUsageRecMachineCreate[0].ACCOUNT_ID__c);
		            			outputCSVrow.add(ZuoraUOM);
		            			outputCSVrow.add(quantityUsed);
		            			outputCSVrow.add(parsedDate);
		            			outputCSVrow.add(endDateTime);
		            			outputCSVrow.add(subId);
		            			outputCSVrow.add(chargeId);
		            			outputCSVrow.add(description);
		            			outputCSVrow.add(dataCenter);
		            			outputCSVrow.add(createdMachineDateTimeString);
		            			outputCSVrow.add(createdZoneDestroyDateTimeString);			            			
		            			outputCSVrow.add(createdUsageRecMachineCreate[0].MACHINE_ID__c);
			            		outputCSVrow.add('1');
			            		system.debug('outputCSVrow for machine_create to zone_destroy = ' + outputCSVrow);
			            		
//			            		system.debug(outputCSVrow[0]);
//			            		system.debug(outputCSVrow[1]);
//			            		system.debug(outputCSVrow[2]);
//			            		system.debug(outputCSVrow[3]);
//			            		system.debug(outputCSVrow[4]);
//			            		system.debug(outputCSVrow[5]);
//			            		system.debug(outputCSVrow[6]);
//			            		system.debug(outputCSVrow[7]);
//			            		system.debug(outputCSVrow[8]);
//			            		system.debug(outputCSVrow[9]);
//			            		system.debug(outputCSVrow[10]);
//			            		system.debug(outputCSVrow[11]);
//			            		system.debug(outputCSVrow[12]);
	            				
	            				outputFile.add(outputCSVrow);
			            			system.debug('outputFile = ' + outputFile);
			            			system.debug('outputFile size = ' + outputFile.size());
			            			
			            			if(outputFile.size() > 1){
			            				for (Integer n=0; n < outputFile.size(); n++){
			            					system.debug('outputFile[' + n + '] = ' + outputFile[n]);
			            				}
			            			}else if(outputFile.size() == 1){
			            				system.debug('outputFile[0] = ' + outputFile[0]);
			            			}
	            				//2 - Calculate usage quantity between zone_resize date and hvm_destroy (if it exists) OR current date 
	            				
							}
							
							List<String> outputCSVrow2 = new List<String>();
							//if hvm destroy record exists for machine id, calculate usage between zone_resize date and hvm_destroy date
							if(createdUsageRecHVMDestroy.size() > 0 && createdUsageRecMachineCreate.size()>0){
								
								//=================================================================================================================
								system.debug('===============Machine create exists and so does Hvm Destroy========================');
								system.debug('row = ' + row);
								String createdDateTimeS_zoneResize = row.DATE_TIME__c;
	            				system.debug('createdDateTimeS_zoneResize = ' + createdDateTimeS_zoneResize);
	            			
	            				String destroyedDateTimeS_zoneResize = createdUsageRecHVMDestroy[0].DATE_TIME__c;
	            				system.debug('destroyedDateTimeS_zoneResize = ' + createdUsageRecHVMDestroy[0].DATE_TIME__c);
	            			
	            				//EXAMPLE DATE: 2013-08-03T13:40:09+00:00 
	            				//EX FORMAT:    YYYY-MM-DDThh:mm:ss+hh:mm            			
	            				Datetime createdDateTime_zoneResize = parseDateTime(createdDateTimeS_zoneResize);
	            				system.debug('createdDateTime_zoneResize = ' + createdDateTime_zoneResize);
	            				String createdDateTimeString_zoneResize = String.valueOf(createdDateTime_zoneResize);
	            			            			
	            				DateTime destroyedDateTime_zoneResize = parseDateTime(destroyedDateTimeS_zoneResize);
	            				system.debug('destroyedDateTime_zoneResize = ' + destroyedDateTime_zoneResize);
	            				String destroyedDateTimeString_zoneResize = String.valueOf(destroyedDateTime_zoneResize);
	            			
	            				String createdUsageRecString_zoneResize = createdUsageRecHVMDestroy[0].MACHINE_ID__c;
	            				system.debug('createdUsageRecHVMDestroy[0].MACHINE_ID__c (createdUsageRecString_zoneResize) = ' + createdUsageRecHVMDestroy[0].MACHINE_ID__c);
	            				
	            				//calculate the quantity from the dates for the machineID specified
	            				Double qUnitsUsed = getQuantity(createdDateTime_zoneResize, destroyedDateTime_zoneResize, createdUsageRecString_zoneResize);
	            				quantity = qUnitsUsed;
	            				String qStringForHVM = String.valueOf(quantity);
	            			
	            				//outputCSVrow[0] = ;
	            				//system('first line of output is ' + );
	            			
	            				//Always set the Usage.StartDate to the last day of the current month (so if today is July 7th, we want this to be 7/31/2013)
	            				//     			datetime myDateTime = datetime.now();
	            				
	            				List <String> queryJoyentSize = new List<String>();
	            				//queryJoyentSize = row[4].split('v');
	            				    
	            				        			
	               				listoftables = 
	            				[SELECT Id, Zuora_UOM__c 
	            				FROM LB_Lookup_Table__c 
	            				WHERE Joyent_Size__c =: row.JOYENT_SIZE__c
	            				//AND Joyent_OS__c =: row[5] 
	            				AND JOYENT_OS2__c =: 'v' LIMIT 1];
	            				
	            			      
	            				if(listoftables.size()>0){
		            				ZuoraUOM = listoftables[0].Zuora_UOM__c;
		            				system.debug('ZuoraUOM = ' + ZuoraUOM);
	            				}
	            			
	            			           			
	            				system.debug('listoftables = ' + listoftables);
	            				
	            				Date obj = Date.today();
	            				system.debug(obj);
	            				
	            				Integer year = obj.year();
	            				system.debug('obj.year = ' + year);
	            				
	            				Integer month = obj.month();
	            				Integer numDays = date.daysInMonth(year, month);
	            				String parsedDate = '';
	            				
	            				if(month<10){
	            					parsedDate = '0' + month + '/' + numDays + '/' + year;
	            					system.debug('parsedDate = ' + parsedDate);
	            				}else{
	            					parsedDate = month + '/' + numDays + '/' + year;
	           						system.debug(parsedDate);
	           					}
	            					
	           					outputCSVrow2.add(createdUsageRecMachineCreate[0].ACCOUNT_ID__c);
		            			outputCSVrow2.add(ZuoraUOM);
		            			outputCSVrow2.add(qStringForHVM);
		            			outputCSVrow2.add(parsedDate);
		            			outputCSVrow2.add(endDateTime);
		            			outputCSVrow2.add(subId);
		            			outputCSVrow2.add(chargeId);
		            			outputCSVrow2.add(description);
		            			outputCSVrow2.add(dataCenter);
		            			outputCSVrow2.add(createdDateTimeString_zoneResize);
		            			outputCSVrow2.add(destroyedDateTimeString_zoneResize);			            			
		            			outputCSVrow2.add(row.ACCOUNT_ID__c);
			            		outputCSVrow2.add('1');
			            		system.debug('outputCSVrow2 = ' + outputCSVrow2);
			            		
			           			outputFile.add(outputCSVrow2);
			            			system.debug('outputFile = ' + outputFile);
			            			system.debug('outputFile size = ' + outputFile.size());
			            			
			            			if(outputFile.size() > 1){
			            				for (Integer n=0; n < outputFile.size(); n++){
			            					system.debug('outputFile[' + n + '] = ' + outputFile[n]);
			            				}
			            			}else if(outputFile.size() == 1){
			            				system.debug('outputFile[0] = ' + outputFile[0]);
			            			}
								
								//=================================================================================================================
								system.debug('HVM_DESTROY date exists for this machine ID.');
								
							}else if(createdUsageRecHVMDestroy.size()==0 && createdUsageRecMachineCreate.size()>0){
								
								system.debug('====================Machine Create Exists, HVM_DESTROY DATE DOES NOT EXIST====================');
								system.debug('row = ' + row);
								String createdDateTimeS_resizeNoDestroy = row.DATE_TIME__c;
	            				system.debug('createdDateTimeS_resizeNoDestroy = ' + createdDateTimeS_resizeNoDestroy);
	            			
	            				Date currentDate = Date.today();
	            				Integer cyear = currentDate.year();
	            				system.debug('currentDate.year = ' + cyear);
	            				
	            				Integer cmonth = currentDate.month();
	            				
	            				Integer cday = currentDate.day();
	            				
	            				String currentDateString = '';
	            				
	            				if(cmonth<10){
	            					currentDateString =  cyear + '-' + '0' + cmonth + '-' + cday + ' ' + '00' + ':' + '00' + ':' + '00';
	            					system.debug('currentDateString = ' + currentDateString);
	            				}else{
	            					currentDateString = cyear + '-' + cmonth + '-' + cday + ' ' + '00' + ':' + '00' + ':' + '00';
	           						system.debug('currentDateString = ' + currentDateString);
	           					}
	           					
	           					
	            				String destroyedDateTimeS_resizeNoDestroy = currentDateString;
	            				system.debug('destroyedDateTimeS_resizeNoDestroy = ' + currentDateString);
	            			
	            				//EXAMPLE DATE: 2013-08-03T13:40:09+00:00 
	            				//EX FORMAT:    YYYY-MM-DDThh:mm:ss+hh:mm            			
	            				Datetime createdDateTime_resizeNoDestroy = parseDateTime(createdDateTimeS_resizeNoDestroy);
	            				system.debug('createdDateTime_resizeNoDestroy = ' + createdDateTime_resizeNoDestroy);
	            				String createdDateTimeString_resizeNoDestroy = String.valueOf(createdDateTime_resizeNoDestroy);
	            			            			
	            				DateTime destroyedDateTime_resizeNoDestroy = parseDateTime(destroyedDateTimeS_resizeNoDestroy);
	            				system.debug('destroyedDateTime_resizeNoDestroy = ' + destroyedDateTime_resizeNoDestroy);
	            				String destroyedDateTimeString_resizeNoDestroy = String.valueOf(destroyedDateTime_resizeNoDestroy);
	            			
	            				String createdUsageRecString_resizeNoDestroy = 'asdf';
	            				system.debug('MACHINE_ID__c (createdUsageRecString_resizeNoDestroy) = ' + row.MACHINE_ID__c);
	            				
	            				//calculate the quantity from the dates for the machineID specified
	            				Double unitsUsed = getQuantity(createdDateTime_resizeNoDestroy, destroyedDateTime_resizeNoDestroy, createdUsageRecString_resizeNoDestroy);
	            				quantity = unitsUsed;
	            				String qString = String.valueOf(quantity);
	            			
	            				//outputCSVrow[0] = ;
	            				//system('first line of output is ' + );
	            			
	            				//Always set the Usage.StartDate to the last day of the current month (so if today is July 7th, we want this to be 7/31/2013)
	            				//     			datetime myDateTime = datetime.now();
	            				
	            				List <String> queryJoyentSizeB = new List<String>();
	            				//queryJoyentSizeB = row[4].split('v');

	            				listoftables = 
	            				[SELECT Id, Zuora_UOM__c 
	            				FROM LB_Lookup_Table__c 
	            				WHERE Joyent_Size__c =: row.JOYENT_SIZE__c
	            				//AND Joyent_OS__c =: row[5] 
	            				AND JOYENT_OS2__c =: 'v' LIMIT 1];
	            				
	            			      
	            				if(listoftables.size()>0){
		            				ZuoraUOM = listoftables[0].Zuora_UOM__c;
		            				system.debug('ZuoraUOM = ' + ZuoraUOM);
	            				}
	            			
	            			           			
	            				system.debug('listoftables = ' + listoftables);
	            				
	            				Date obj = Date.today();
	            				system.debug(obj);
	            				
	            				Integer year = obj.year();
	            				system.debug('obj.year = ' + year);
	            				
	            				Integer month = obj.month();
	            				Integer numDays = date.daysInMonth(year, month);
	            				String parsedDate = '';
	            				
	            				if(month<10){
	            					parsedDate = '0' + month + '/' + numDays + '/' + year;
	            					system.debug('parsedDate = ' + parsedDate);
	            				}else{
	            					parsedDate = month + '/' + numDays + '/' + year;
	           						system.debug(parsedDate);
	           					}
	            					
	           					outputCSVrow2.add(createdUsageRecMachineCreate[0].ACCOUNT_ID__c);
		            			outputCSVrow2.add(ZuoraUOM);
		            			outputCSVrow2.add(qString);
		            			outputCSVrow2.add(parsedDate);
		            			outputCSVrow2.add(endDateTime);
		            			outputCSVrow2.add(subId);
		            			outputCSVrow2.add(chargeId);
		            			outputCSVrow2.add(description);
		            			outputCSVrow2.add(dataCenter);
		            			outputCSVrow2.add(createdDateTimeString_resizeNoDestroy);
		            			outputCSVrow2.add(destroyedDateTimeString_resizeNoDestroy);			            			
		            			outputCSVrow2.add(row.MACHINE_ID__c);
			            		outputCSVrow2.add('1');
			            		system.debug('outputCSVrow2 = ' + outputCSVrow2);
			            		
			           			outputFile.add(outputCSVrow2);
			            			system.debug('outputFile = ' + outputFile);
			            			system.debug('outputFile size = ' + outputFile.size());
			            			
			            			if(outputFile.size() > 1){
			            				for (Integer n=0; n < outputFile.size(); n++){
			            					system.debug('outputFile[' + n + '] = ' + outputFile[n]);
			            				}
			            			}else if(outputFile.size() == 1){
			            				system.debug('outputFile[0] = ' + outputFile[0]);
			            			}
							} //902
	            		} //640
	            		
        forLoopCounter++;
        } //336
	

    	
    	
//=====================BEGIN POPULATING RECORDS=================================================================

		system.debug('BEGIN POPULATING RECORDS===================================================================');
		Integer counter = 1;
		system.debug('outputFile is ' + outputFile);
		system.debug('outputFile Size: ' + outputFile.size());
		
		List<Zuora.zObject> usageRecList = new List<Zuora.zObject>();
	 	for(List<String> oFileRow :  outputFile){
	 		zuora.zobject usageRec = new zuora.zobject('Usage');
	 		system.debug('oFileRow is ' + oFileRow);
	 		system.debug('Counter is ' + counter);
	 		
	 		system.debug('AccountNumber = ' + oFileRow[0]);
	 		system.debug('UOM = ' + oFileRow[1]);
	 		system.debug('Quantity = ' + oFileRow[2]);
	 		system.debug('StartDateTime = ' + oFileRow[3]);
	 		system.debug('EndDateTime = ' + oFileRow[4]);
	 		system.debug('SubscriptionId = ' + oFileRow[5]);
	 		system.debug('ChargeId = ' + oFileRow[6]);
	 		system.debug('Description = ' + oFileRow[7]);
	 		system.debug('Datacenter__c = ' + oFileRow[8]);
	 		system.debug('Joyentcreatetime__c = ' + oFileRow[9]);
	 		system.debug('Joyentendtime__c = ' + oFileRow[10]);
	 		system.debug('MachineId__c = ' + oFileRow[11]);
	 		system.debug('BatchId__c = ' + oFileRow[12]);
	 		//09/30/2013
	 		Integer length = oFileRow[3].length();
	 		string year = oFileRow[3].substring(6,length);
	 		system.debug(year);
			string month = oFileRow[3].substring(0,2);
			system.debug(month);
			string day = oFileRow[3].substring(3,5);
			system.debug(day);
			String hour = '00';
			String minute = '00';
			String second = '00';
	        String parsedDate = year + '-' + month + '-' + day + 'T' + hour + ':' + minute + ':' + second;
	        system.debug('parsedDate = '+ parsedDate + '. Size is '+ parsedDate.length());
	        Date myDate = date.valueOf(parsedDate); 
	        system.debug('myDate = ' + myDate);
	        String tempDate = Zuora.zUtil.getZuoraDate(myDate);
	        system.debug('====================ZUORA DATE IS ' + tempDate);
	 		
	        usageRec.setValue('AccountNumber', oFileRow[0]);
	        system.debug('usageRec = ' + usageRec);
	        
	        usageRec.setValue('UOM', oFileRow[1]);
	        system.debug('usageRec = ' + usageRec);
	        
	        usageRec.setValue('Quantity', oFileRow[2]);
	        system.debug('usageRec = ' + usageRec);
	        
	        usageRec.setValue('StartDateTime', tempDate);
	        system.debug('usageRec = ' + usageRec);
	        
//	        usageRec.setValue('EndDateTime', oFileRow[4]);
//	        system.debug('usageRec = ' + usageRec);
	        
	        if(oFileRow[5].length() > 0){
	        	usageRec.setValue('SubscriptionId', oFileRow[5]);
	        	system.debug('usageRec = ' + usageRec);
	        }
	        if(oFileRow[6].length() > 0){
	        	usageRec.setValue('ChargeId', oFileRow[6]);
	        	system.debug('usageRec = ' + usageRec);
	        }
	        
	        usageRec.setValue('Description', oFileRow[7]);
	        system.debug('usageRec = ' + usageRec);
	        
	        usageRec.setValue('Datacenter__c', oFileRow[8]);
	        system.debug('usageRec = ' + usageRec);
	        
	        usageRec.setValue('Joyentcreatetime__c', oFileRow[9]);
	        system.debug('usageRec = ' + usageRec);
	        
	        usageRec.setValue('Joyentendtime__c', oFileRow[10]);
	        system.debug('usageRec = ' + usageRec);
	        
	        usageRec.setValue('MachineId__c', oFileRow[11]);
	        system.debug('usageRec = ' + usageRec);
	        
	        usageRec.setValue('BatchId__c', oFileRow[12]);
	        system.debug('usageRec = ' + usageRec);
	        
	        usageRecList.add(usageRec);
	        system.debug('usageRecList = ' + usageRecList);
	        
	        
	
		counter++;
	 	}

//======================================BEGIN SAVING RECORDS===================================================
	 	
	 	system.debug('Begin SendToZuora Save Process');
	 	Zuora.zApi zApiInstance = new Zuora.zApi();

	        
	        try {
	        	
	        	//remember that Z-360 will use the credentials in the SF tenant. Comment out the following lines if this has been done.
	        	
				zApiInstance.setEndpoint('https://apisandbox.zuora.com/apps/services/a/45.0');
				//zApiInstance.setEndpoint('https://www.zuora.com/apps/services/a/45.0');
				zApiInstance.zlogin('david.goossens@z-linkbermuda.com', 'Zour4Link$');
				//zApiInstance.zlogin();
				
			} catch (Zuora.zRemoteException ex) {
				if ('INVALID_LOGIN' == ex.code) { 
					// The login credentials are invalid.
					system.debug('Incorrect Login.');
				}
				system.debug('login exception: ' + ex.getmessage());
				return;
			}
	 			system.debug('usageRecList size = ' + usageRecList.size());
				List<Zuora.zApi.SaveResult> results = zApiInstance.zcreate(usageRecList);
				system.debug('results ' + results);
				
			Integer saveUsageCounter = 1;
			String printResult ='';
			
	 		for (Zuora.zApi.SaveResult result : results) {
					if (result.Success){
						//get the newly created id
						String createdId = result.Id;
						system.debug('result.Id = ' + result.Id);
						printResult = saveUsageCounter + ' Usage Record(s) created in Zuora';
						saveUsageCounter = saveUsageCounter + 1;
				}else{
					//failure
					Zuora.zObject[] errors = result.errors;
					for (Zuora.zObject error : errors) {
						String errorCode = (String)error.getValue('Code');
						String message = (String)error.getValue('Message');
						system.debug('errorCode = ' + errorCode);
						system.debug('message = ' + message);
				}
			}
			
		}
	}
}
