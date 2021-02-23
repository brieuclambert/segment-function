const fetch = require("node-fetch");

async function createRecord(auth, userId, email) {
	var data = {
		Livestorm_User_Id__c: userId,
		Email: email,
		lastName: email.split('@')[0],
		company: email.split('@')[1],
		LeadSource: 'Signup'
	};
	let options = {
		method: 'POST',
		headers: {
			'Content-Type': 'application/json',
			Authorization: 'Bearer ' + auth.access_token
		},
		body: JSON.stringify(data)
	};

	let response = await fetch(
		'https://livestorm3.my.salesforce.com/services/data/v50.0/sobjects/Lead',
		options
	);
	return response;
}

async function searchContact(options, email) {
	console.log('starting search');
	var search = encodeURI(
		'FIND {' +
			email +
			'} IN EMAIL FIELDS RETURNING Lead(FirstName,LastName, Id, Livestorm_User_Id__c), Contact(FirstName,LastName, Id, Livestorm_User_Id__c)'
	);
	var endpoint =
		'https://livestorm3.my.salesforce.com/services/data/v50.0/search/?q=' +
		search;

	let response = await fetch(endpoint, options);
	return response.json();
}

async function getAccessToken(settings) {
	let data =
		'grant_type=password&client_id=' +
		settings.clientId +
		'&client_secret=' +
		settings.clientSecret +
		'&username=' +
		settings.username +
		'&password=' +
		settings.password;
	let options = {
		method: 'POST',
		headers: {
			'Content-Type': 'application/x-www-form-urlencoded'
		},
		body: data
	};

	let response = await fetch(
		'https://livestorm3.my.salesforce.com/services/oauth2/token',
		options
	);
	return response.json();
}

async function updateRecord(record, auth, userId) {
	var data = {
		Livestorm_User_Id__c: userId
	};
	let options = {
		method: 'PATCH',
		headers: {
			'Content-Type': 'application/json',
			Authorization: 'Bearer ' + auth.access_token
		},
		body: JSON.stringify(data)
	};

	let response = await fetch(
		'https://livestorm3.my.salesforce.com/' + record.attributes.url,
		options
	);
	return response;
}

async function onTrack(event, settings) {
	if (event.event == 'Signed up') {
		auth = await getAccessToken(settings);
		let options = {
			method: 'GET',
			headers: {
				'Content-Type': 'application/json',
				Authorization: 'Bearer ' + auth.access_token
			}
		};
		var userId = event.userId;
		var email = event.properties.email;
		console.log(email);
		searchContact(options, email).then(response => {
			console.log(response.searchRecords);
			var contact = response.searchRecords.find(
				record => record.attributes.type == 'Contact'
			);
			var lead = response.searchRecords.find(
				record => record.attributes.type == 'Lead'
			);
			if (contact) {
				console.log(contact);
				updateRecord(contact, auth, userId).then(response =>
					console.log(response.status + ' - ' + response.statusText)
				);
			} else if (lead) {
				console.log(lead);
				updateRecord(lead, auth, userId).then(response =>
					console.log(response.status + ' - ' + response.statusText)
				);
			} else {
				console.log('creating lead');
				createRecord(auth, userId, email).then(response =>
					console.log(response)
				);
			}
		});
	}
}

var event = {
  "channel": "server",
  "context": {
    "library": {
      "name": "analytics-ruby",
      "version": "2.2.8"
    },
    "protocols": {
      "sourceId": "******"
    }
  },
  "event": "Signed up",
  "integrations": {},
  "messageId": "5ec610c6-***-431d-***-846d7cff4d55",
  "originalTimestamp": "2021-02-23T12:13:41.215+00:00",
  "projectId": "*******",
  "properties": {
    "email": "b***.v***@g***.c***",
    "source_url": "url/#/google-signup",
    "type": "google"
  },
  "receivedAt": "2021-02-23T12:13:42.142Z",
  "sentAt": "2021-02-23T12:13:41.707Z",
  "timestamp": "2021-02-23T12:13:41.650Z",
  "type": "track",
  "userId": "****-8e62-***-a499-c5906a704539",
  "version": 2,
  "writeKey": "***********"
}

var settings = {
	clientId: "myclientId",
	clientSecret: "myClientSecret",
	username: "myUsername",
	password: "myPassword"
}

onTrack(event, settings)
