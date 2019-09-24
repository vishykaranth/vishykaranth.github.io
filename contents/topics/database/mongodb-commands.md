# Create-Read-Update-Delete (CRUD)

You can use normal MongoDB commands to create, read and modify documents.  In this example we will use the MongoDB command line shell (available from [mongodb.com](https://www.mongodb.com/download-center/community)). 

For starters, let's create a few documents:

```javascript
mongo > db.tutorial.insertMany([{name:"Guy",rating:10},{name:"Mike",rating:9}]);
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("5c737dd2c8bdf9bf3676fee9"),
		ObjectId("5c737dd2c8bdf9bf3676feea")
	]
}
```
We can query the documents as usual: 

```javascript
mongo >  db.tutorial.find({name:"Guy"});
{ "_id" : ObjectId("5c737dd2c8bdf9bf3676fee9"), "name" : "Guy", "rating" : 10 }
```

Updating the documents works exactly as it would in MongoDB:

```javascript
mongo > db.tutorial.update({"name":"Guy"},{$inc:{rating:1}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

mongo > db.tutorial.update({"name":"Mike"},{$inc:{rating:-1}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

mongo > db.tutorial.find({});
{ "_id" : ObjectId("5c737dd2c8bdf9bf3676fee9"), "name" : "Guy", "rating" : 11 }
{ "_id" : ObjectId("5c737dd2c8bdf9bf3676feea"), "name" : "Mike", "rating" : 8 }
```

# Versions

So far so good.  However, under the hood things worked very differently from MongoDB.  When the updates were issued, they did not obliterate the old versions of data.  Instead, they created new _versions_ of the data.  

We can see what version we are on with this command:

```javascript
mongo> db.runCommand({getVersion:1});
{
        "ok" : 1,
        "response" : "The version is set to: 'current'",
        "version" : NumberLong(10206),
        "status" : "current"
}
```

We are on version 10206.  If we do anther update we'll see our version increment to 10207:

```javascript
mongo> db.tutorial.update({"name":"Guy"},{$inc:{rating:2}});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
mongo> 
mongo> db.tutorial.find({name:"Guy"});
{ "_id" : ObjectId("5c737e67c8bdf9bf3676feeb"), "name" : "Guy", "rating" : 15 }
mongo>
mongo> db.runCommand({getVersion:1});
{
        "ok" : 1,
        "response" : "The version is set to: 'current'",
        "version" : NumberLong(10207),
        "status" : "current"
}
```
If we set our version back to 10206, we can see the contents of the database as it was before the last update:

```javascript
mongo> db.tutorial.find({name:"Guy"});
{ "_id" : ObjectId("5c737e67c8bdf9bf3676feeb"), "name" : "Guy", "rating" : 15 }
mongo> db.runCommand({setVersion:10206});
{
        "ok" : 1,
        "response" : "The version has been set to: '10206'",
        "version" : NumberLong(10206),
        "status" : "userDefined"
}
mongo> db.tutorial.find({name:"Guy"});
{ "_id" : ObjectId("5c737e67c8bdf9bf3676feeb"), "name" : "Guy", "rating" : 13 }
mongo> db.runCommand({setVersion:'current'});

```
Note that we set the version back to 'current' after retrieving the old information.  This is good practice, since unless the version is set to 'current', modifications to the database will be denied because you can't change the past. 

# Document history 

We can retrieve a complete history of a document (or documents) using the `docHistory` command.  
```javascript
mongo> db.runCommand({docHistory:{collection:"tutorial",filter:{name:"Guy"}}});
{
        "ok" : 1,
        "docHistory" : [
                {
                        "collection" : "tutorial",
                        "_id" : ObjectId("5c737e67c8bdf9bf3676feeb"),
                        "history" : {
                                "versions" : [
                                        {
                                                "minVersion" : NumberLong(10206),
                                                "maxVersion" : NumberLong(10206),
                                                "status" : "Unproven",
                                                "started" : "2019-02-25 04:34:32",
                                                "ended" : "2019-02-25 04:34:32",
                                                "document" : {
                                                        "name" : "Guy",
                                                        "rating" : 13
                                                }
                                        },
                                        {
                                                "minVersion" : NumberLong(10207),
                                                "maxVersion" : NumberLong("9223372036854775807"),
                                                "status" : "Unproven",
                                                "started" : "2019-02-25 04:39:24",
                                                "ended" : "2019-02-25 04:39:24",
                                                "document" : {
                                                        "name" : "Guy",
                                                        "rating" : 15
                                                }
                                        }
                                ]
                        }
                }
        ]
}
```
Under the hood, ProvenDB is adding metadata to each document that defines the scope of the current version of the document.  This metadata can be revealed with the [showMetadata](doc:showmetadata) command:

```javascript
mongo> db.runCommand({showMetadata:true});
{ "ok" : 1 }
mongo> db.tutorial.find({name:"Guy"}).pretty();
{
        "_id" : ObjectId("5c737e67c8bdf9bf3676feeb"),
        "_provendb_metadata" : {
                "_id" : ObjectId("5c737e67c8bdf9bf3676feeb"),
                "_mongoId" : ObjectId("5c737e6862ad840be62d0445"),
                "minVersion" : NumberLong(10206),
                "hash" : "f6138f36f4c6140c36715790cf06c4c565330545fb210d146778df838560ec8a",
                "maxVersion" : NumberLong(10206)
        },
        "name" : "Guy",
        "rating" : 13
}
```

Most of the time you don't need to worry about the metadata, but it's there if you want to try and understand what's going on. 

# Blockchain proofs

So we've seen how versions work, but how does this relate to the Blockchain?   Well, versions in ProvenDB may be anchored to the Blockchain by placing a hash of the versions content on the Blockchain.  There's some complex internals using _Merkle Trees_ involved in this process which you can read about in the [Concepts](doc:concepts) manual.  For now, let's just see how you do it.

You create a Blockchain proof for a specific database version using the `submitProof` command.  Let's create a proof for version 10206.  

```javascript
mongo> db.runCommand({submitProof:10206})
{
        "ok" : 1,
        "version" : NumberLong(10206),
        "dateTime" : ISODate("2019-02-25T05:47:06Z"),
        "hash" : "2a161f14b1431c062f20a62ba229f97d9f92a848effc2438d1aeb2b468d5760c",
        "proofId" : "c8a4f180-38c0-11e9-b4b3-01d13dea962c",
        "status" : "Pending"
}
```
Proofs take a while to get on the Blockchain.  When first submitted they are 'Pending'. 

You can check the proof using the [getProof](doc:getproof) command:

```javascript
mongo>  db.runCommand({getProof:10207});
{
	"ok" : 1,
	"proofs" : [
		{
			"proofId" : "986c0390-38c1-11e9-b4b3-01034e52777e",
			"version" : NumberLong(10207),
			"submitted" : ISODate("2019-02-25T05:52:54Z"),
			"type" : "Full",
			"hash" : "401aa031471f5f9badb076600dafc89ff2baa88119d8ab48bcc5a410b10cae41",
			"status" : "Pending",
```

After a few seconds they make their way onto the [chainpoint](chainpoint.org) network - see the [Concepts](doc:concepts) manual - and the status changes to 'Submitted':

```javascript
mongo> db.runCommand({getProof: 10207});
{
	"ok" : 1,
	"proofs" : [
		{
			"proofId" : "986c0390-38c1-11e9-b4b3-01034e52777e",
			"version" : NumberLong(10207),
			"submitted" : ISODate("2019-02-25T05:52:54Z"),
			"type" : "Full",
			"hash" : "401aa031471f5f9badb076600dafc89ff2baa88119d8ab48bcc5a410b10cae41",
			"status" : "Submitted",
			"details" :

```

Eventually they make their way onto the bitcoin Blockchain and are 'Valid'.  We can also see the bitcoin block and transaction ID that includes our proof.  

```javascript
mongo> db.runCommand({getProof:2176,format:'binary'})
{
	"ok" : 1,
	"proofs" : [
		{
			"proofId" : "09f8a160-2feb-11e9-b4b3-01d156d5b86a",
			"version" : NumberLong(2176),
			"submitted" : ISODate("2019-02-13T23:56:54Z"),
			"type" : "Full",
			"hash" : "587e98d9c567073bcf0a059f88c9c093f9a8bcb5164fee5ca023550d1fce2fab",
			"status" : "Valid",
			"details" : {
				"protocol" : {
					"name" : "chainpoint",
					"uri" : "http://13.238.131.2",
					"hashIdNode" : "09f8a160-2feb-11e9-b4b3-01d156d5b86a",
					"chainpointLocation" : "https://b.chainpoint.org/calendar/2710061/data"
				},
				"collections" : [ ... ],
				"btcTxn" : "345d7f9a846fcf443e438410cd9d7240b66fd2bb9aecfeacb3e82d5fef337d49",
				"btcBlock" : "562935"
			},
			"proof" : BinData(0,"eJyVVj1vY ... 330=")
		}
	]
}
```

Above  we are using the 'binary' format to produce a compact though unreadable format.  We could also use the JSON format in which case we will see see a [chainpoint](chainpoint.org) compatible JSON document containing the Merkle tree path that associates the hash of our database version with the Blockchain transaction.   In either case, we can pass this proof to the Chainpoint API or command line tools to validate the proof.  

 We can retrieve a proof for a single document with the [getDocumentProof](doc:getdocumentproof)  command:

```javascript
db.runCommand({
    getDocumentProof  : {
         collection:"tutorial",
         provendbId:ObjectId("5bcebc1d0298522dfa857d65"),
         versionId:2162
    }
});
```

 

The [verifyProof](doc:verifyproof) command is similar to the [getProof](doc:getproof) command, but it recalculates all hash values and reconstructs the merkle tree proof.  This ensures that the proof is still valid - i.e, that no-one has tampered with the underlying data. 

```javascript
db.runCommand({validateProof:proofId});
{
	"versionId" : NumberLong("d07d65b0-d68a-11e8-b4b3-01f02cb25859"),
	"dateTime" : ISODate("2018-10-23T21:53:50.817Z"),
	"hash" : "1768ede09c957a66b2ad22c1b826cf252ebb3d31ebc19f41802f322ed099dbf1",
	"proofId" : "d07d65b0-d68a-11e8-b4b3-01f02cb25859",
	"proofStatus" : "Valid",
	"proof" : {
		"@context" : "https://w3id.org/chainpoint/v3",
		"type" : "Chainpoint",
		"hash" : "1768ede09c957a66b2ad22c1b826cf252ebb3d31ebc19f41802f322ed099dbf1",
		"hash_id_node" : "d07d65b0-d68a-11e8-b4b3-01f02cb25859",
		"hash_submitted_node_at" : "2018-10-23T06:13:52Z",
		"hash_id_core" : "d13a4d10-d68a-11e8-a51f-014e49f71db3",
		"hash_submitted_core_at" : "2018-10-23T06:13:53Z",
		"branches" : [
			{
                // Lots of merkle path information here 
			}
		]
	}
}
```
# Other features

This tutorial introduces ProvenDB and illustrates the core MongoDB-compatible API.  There are other features that are the subject of their own sections:

* ** [Compacting](doc:compact)  ** unproven versions of the database to save space
* ** [Exporting ](doc:exporting-your-data) ** documents and proofs and validating these using our open source command line validation tool 
* ** [Forgetting](doc:forgetting)  ** documents to be GDPR compliant
* Using the ProvenDB powered ** [ProvenDocs](doc:what-is-provendocs)  ** application