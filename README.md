# blob_md5_checksum
Calculate the MD5 checksum for a Oracle BLOB. This calculation matches the Content MD5 Hash 
that one finds in the file properties at Oracle Cloud Infrastructure Storage (OCI Storage or OCI Bucket).

For those need an MD5 checksum of a string (varchar2), please take a look at an APEX Utility
https://docs.oracle.com/en/database/oracle/application-express/20.2/aeapi/MD5_CHECKSUM-Function.html#GUID-025DF33F-CC93-4702-BB6E-453397BD7195

I could not find an similar function for MD5 hash for BLOB.
I wanted to calculate the MD5 hash for a blob prior to the upload to OCI so we can compare the
initial calculation with the value returned from OCI in the response headers. If the two
values match, then we can be assured that the BLOB contents at both ends of the transfer
match.

To capture the response headers, you'll need to adopt code that loops through
APEX_WEB_SERVICE.G_HEADERS
Not much has been provided on the Oracle documentation about this array. Here are two
articles I found (JULY 2021)
https://docs.oracle.com/en/database/oracle/application-express/20.2/aeapi/Retrieving-Cookies-and-HTTP-Headers.html#GUID-39ADA850-DDD1-4A71-834A-1F7BC4FB2269
https://docs.oracle.com/en/database/oracle/application-express/20.2/aeapi/MAKE_REST_REQUEST-Procedure-Signature-1.html#GUID-8E618B80-FDE9-475C-BC0C-B7C985FCECFF

The sample code below captures the OCI calcuated MD5.

We tend to capture the request headers and response headers in an API Staging table. When APIs go well, they do so very well.
When they mess up, you need all of the tools and information you can find.

procedure response_headers (
	r_bucket_object			in out api_bucket_object%rowtype,
	r_staging						in out api_staging%rowtype
	)
as
begin
	for i in 1.. apex_web_service.g_headers.count loop
		r_staging.response_headers := r_staging.response_headers || apex_web_service.g_headers(i).name||':';
		r_staging.response_headers := r_staging.response_headers || apex_web_service.g_headers(i).value || lf;
		case 
			when apex_web_service.g_headers(i).name = 'etag' then
				r_bucket_object.object_etag		:= apex_web_service.g_headers(i).value;
			when apex_web_service.g_headers(i).name = 'opc-content-md5' then				
				r_bucket_object.remote_content_md5	:= apex_web_service.g_headers(i).value;
				--r_bucket_object.initial_content_md5	:= apex_web_service.g_headers(i).value;
			when apex_web_service.g_headers(i).name = 'version-id' then	
				r_bucket_object.object_version_id 	:= apex_web_service.g_headers(i).value;
			else
				null;
		end case;
	end loop;	
end response_headers;



