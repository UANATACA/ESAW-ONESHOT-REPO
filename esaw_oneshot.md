# What it is

<div style="text-align: justify">
</div>

# How it works



# Configuration



# Endpoint URLs

To use eSAW One-Shot API, you will need the following hosts:

**For testing environment:**
 - api.sandbox.modernpki.com
 - demo.esignanywhere.net

**For producion environment:**
 - api.modernpki.com
 - saas.esignanywhere.net

# Authentication

We will use two different authentication methods depending on the host that is being called.
For .uanataca domains this is performed providing the server a certificate of an enabled API User. This certificate will be provided by operations department.
This is an example HTTP POST request performed with cURL:

	1 | curl --key key.pem --cert cert.pem -H "Content-Type: application/json" -d @params.json -X POST https://api.sandbox.modernpki.com/api/v1/esawoneshot/

On the other hand, for .esignanywhere domains this is performed by applying an API Key based authentication. The value must be sent along the headers as:

	Key: ApiToken
	Value: 1234fedq

Token value can be generated through the user panel in demo.esignanywhere.net in an autonomous way.

Each environment will have its own accesses, they are not shared in any case.

# Workflow

A common envelope generation process involves the steps shown below. It is assumed that an envelope template on the eSAW platform that contains n signature fields and n recipients is already created.

1) Creation and approval of a request for signer x
2) Document upload to be signed on eSAW platform
3) Fill the envelope template with the previous document id returned in step 2
4) Fill the envelope template signing activity with the token returned in step 3
5) Create and send the envelope

> STEP 1: Creation of a certificate request

API Reference: 

This call must include enough information to identify the requester user. Full description of arguments accepted by this endpoint can be found in the API call detailed documentation.

	curl -i -X POST \
	https://api.sandbox.modernpki.com/api/v1/esawoneshot \
	-H 'Content-Type: application/json' \
	-d '{
		"rao": {
			"username": "12345678",
			"password": "pwd123456",
			"pin": "18s876d4"
		},
		"documents": {
			"document_front": {
				"content_type": "image/png",
				"data": "iVBORw0KGgoAAAANSUhEUgAAAMgAAADICAYAAACtWK6eAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAACxIAAAsSAdLdfvwAAGNGSURBVHhe78rahq0UHe9g/miIt1KvWcEqLtfK5GEdm..."
			},
			"document_rear": {
				"content_type": "image/png",
				"data": "iVBORw0KGgoAAAANSUhEUgAAAMgAAADICAYAAACtWK6eAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAACxIAAAsSAdLdfvwAAGNGSURBVHhe78rahq0UHe9g/miIt1KvWcEqLtfK5GEdm..."
			},
			"owner_photo": {
				"content_type": "image/png",
				"data": "iVBORw0KGgoAAAANSUhEUgAAAMgAAADICAYAAACtWK6eAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAACxIAAAsSAdLdfvwAAGNGSURBVHhe78rahq0UHe9g/miIt1KvWcEqLtfK5GEdm..."
			}
		},
		"request": {
			"profile": "PFnubeNC",
			"registration_authority": 1012,
			"communication_language": "ES",
			"id_document_type": "IDC",
			"id_document_country": "ES",
			"serial_number": "12976587A",
			"given_name": "John",
			"surname_1": "Smith",
			"surname_2": "Smith",
			"email": "test@test.com",
			"mobile_phone_number": "+34666777888"
		}
	}'

Response contains info from the created certificate request such as **esaw_token** that will be needed in further stages

	{
    	"status": "ok",
    	"req_pk": 113929,
    	"esaw_token": "ba4019803aaf2cd0a9755a688c8963901bfef4b942d7d2179e4e1e44c602"
	}

> STEP 2: Upload file

API Reference:

With this call we will upload the documents to be signed through the envelope

Note that this endpoint has to be called for every document that it wants to be signed.

	curl -i -X POST \
	https://demo.esignanywhere.net/Api/v6/file/upload \
	-H 'Content-Type: multipart/form-data' \
	-H 'ApiToken: 1234fedq' \
	-F 'File=@"path/to/document"'

Response contains the **FileId** needed it further stages

	{
    	"FileId": "aa4e3300-c254-4430-ac16-2f2d96090b3a"
	}

> STEP 3: Create and send envelope

API Reference:

An envelope must be created and send to the end user, but previously we have to fill the data accordingly to link the envelope with the certificate request we just created.

	curl -L \
	'https://demo.esignanywhere.net/Api/v6/envelope/send' \
	-H 'Content-Type: application/json' \
	-H 'ApiToken: 1234fedq' \
	--data-raw \
	'{
	"Documents": [
		{
		"FileId": "aa4e3300-c254-4430-ac16-2f2d96090b3a", #FileId from step 2
		"DocumentNumber": 1
		}
	],
	"Name": "dummy.pdf",
	"AddDocumentTimestamp": false,
	"ShareWithTeam": true,
	"LockFormFieldsOnFinish": false,
	"Activities": [
		{
		"Action": {
			"Sign": {
			"RecipientConfiguration": {
				"ContactInformation": {
				"Email": "test@test.com",
				"GivenName": "Test",
				"Surname": "Envelope",
				"LanguageCode": "ES"
				},
				"SendEmails": true,
				"AllowAccessAfterFinish": false,
				"IncludedEmailAppLinks": {
				"Android": false,
				"iOS": false,
				"Windows": false
				},
				"AllowDelegation": true,
				"RequireViewContentBeforeFormFilling": false
			},
			"SequenceMode": "NoSequenceEnforced",
			"Elements": {
				"Signatures": [
				{
					"ElementId": "Firma_9a4557b7-35bf-3f19-d36b-ffe2d4ebd8bb",
					"Required": true,
					"DocumentNumber": 1,
					"UseExternalTimestampServer": false,
					"AllowedSignatureTypes": {
					"SignaturePlugins": [
						{
						"PluginId": "GenericSigningPluginBit4idOneShot",
						"Preferred": false,
						"StampImprintConfiguration": {
							"DisplayExtraInformation": true,
							"FontName": "Times New Roman",
							"FontSizeInPt": 11,
							"DisplayEmail": true,
							"DisplayIp": true,
							"DisplayName": true,
							"DisplaySignatureDate": false
						}
						}
					]
					},
					"FieldDefinition": {
					"Position": {
						"PageNumber": 1,
						"X": 83,
						"Y": 68.91999999999996
					},
					"Size": {
						"Width": 190,
						"Height": 80
					}
					}
				}
				]
			},
			"SignatureDataConfiguration": {
				"SignaturePluginData": [
					{
						"PluginId": "GenericSigningPluginBit4idOneShot",
						"Fields": [
							{
								"Key": "requestToken",
								"Value": "ba4019803aaf2cd0a9755a688c8963901bfef4b942d7d2179e4e1e44c602" #Token from step 1
							}
						]
					}
				]
			},
			"SigningGroup": "1"
			}
		},
		"VisibilityOptions": [
			{
			"DocumentNumber": 1,
			"IsHidden": false
			}
		]
		},
		{
		"Action": {
			"SendCopy": {
			"RecipientConfiguration": {
				"ContactInformation": {
				"Email": "test@test.com",
				"GivenName": "Test",
				"Surname": "Envelope",
				"LanguageCode": "ES"
				}
			},
			"CopyingGroup": "2"
			}
		},
		"VisibilityOptions": [
			{
			"DocumentNumber": 1,
			"IsHidden": false
			}
		]
		}
	],
	"EmailConfiguration": {
		"Subject": "Firme el sobre adjunto",
		"Message": "Estimado #RecipientFirstName# #RecipientLastName#\n\n#PersonalMessage#\n\nFirme el sobre #EnvelopeName#\n\nEl sobre caducará el #ExpirationDate#"
	},
	"ReminderConfiguration": {
		"Enabled": true,
		"FirstReminderInDays": 5,
		"ReminderResendIntervalInDays": 3,
		"BeforeExpirationInDays": 3
	},
	"ExpirationConfiguration": {
		"ExpirationInSecondsAfterSending": 2419200
	}
	}'

Response contains the created EnvelopeId that got sent to the user ready to be signed with One-Shot GSP.

	{
    	"EnvelopeId": "bc6752b7-f761-4b88-91f6-5e3112a716a3"
	}

# Postman collection

A postman collection is available as a support for a quick start.<br>
It is only required to edit `host`variable in Postman environment with the desired domain

<a href="https://cdn.bit4id.com/es/uanataca/public/vol/Uanataca_VOL_Postman.zip"> eSAW One-Shot Postman collection download</a>

<div id="APIReference" style="padding-top: 60px;"><h1>API Reference<h1></div>
