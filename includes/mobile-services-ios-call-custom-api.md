
### Chiamare un'API personalizzata da un'app per iOS

Per chiamare un'API personalizzata da un client iOS, usare il metodo `MSClient invokeAPI`. Sono disponibili due versioni di questo metodo: una per le richieste in formato JSON e una per qualsiasi tipo di dati.

	/// Invokes a user-defined API of the Mobile Service.  The HTTP request and
	/// response content will be treated as JSON.
	-(void)invokeAPI:(NSString *)APIName
	            body:(id)body
	      HTTPMethod:(NSString *)method
	      parameters:(NSDictionary *)parameters
	         headers:(NSDictionary *)headers
	      completion:(MSAPIBlock)completion;

	/// Invokes a user-defined API of the Mobile Service.  The HTTP request and
	/// response content can be of any media type.
	-(void)invokeAPI:(NSString *)APIName
	            data:(NSData *)data
	      HTTPMethod:(NSString *)method
	      parameters:(NSDictionary *)parameters
	         headers:(NSDictionary *)headers
	      completion:(MSAPIDataBlock)completion;


Ad esempio, per inviare una richiesta JSON a un'API personalizzata denominata "sendEmail", passare un oggetto `NSDictionary` per i parametri di richiesta:

	NSDictionary *emailHeader = @{ @"to": @"email.com", @"subject" : @"value" };

	[self.client invokeAPI:@"sendEmail"
	     body:emailBody
	     HTTPMethod:@"POST"
	     parameters:emailHeader
	     headers:nil
	     completion:completion ];
	    
Se sono necessari i dati restituiti, è possibile usare un codice simile al seguente:

	[self.client invokeAPI:apiName
                 body:yourBody
           HTTPMethod:httpMethod
           parameters:parameters
              headers:headers
           completion:  ^(NSData *result,
                          NSHTTPURLResponse *response,
                          NSError *error){
               // error is nil if no error occured
               if(error) { 
                   NSLog(@"ERROR %@", error);
               } else {
                   
		// do something with the result
               }
               
               
           }];

		

<!---HONumber=AcomDC_0204_2016-->