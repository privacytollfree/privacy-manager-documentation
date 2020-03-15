# Privacy Manager Documentation

### Get Started with CCPATollFree Webhooks, the value provider for toll free number for and privacy request manager.  ###

## Webhooks Introduction ##

Webhooks from your privacy manager at ccpatollfree.com allows you to retrieve data from your privacy request in real time securely. We only support webhooks https URL. We do not restrict any cypher suite as we believe it's your responsibility to chose the right https cypher. Our webhooks are triggered based on events happening in the system. For instance, a privacy_request.received sends all data about the privacy request that's available at the time. The voicemail, or transcription may come later. Each event always include the latest state of the object you are about to receive - even under replay. We are not maintaining a list of changes, rather we order every events in the system and send all the data available. You may use the signature timestamp to evaluate when the object was loaded, and it's safe to say that the latest value is always the one with the latest signature timetamps. You should keep that in your record. If you replay an older event, we will fetch the latest state of the object, and the signature timestamp will be the time now, so you are safe to update your record. Note that we are in the process of building a full API, and you may reach out to us if you wish to request something. All request are currently multipart due to the nature of privacy request.

## Security ##

When it comes to webhook, you must validate that the data comes from us. The signature parameters in our request to your endpoint will contain three fields. A random token, a timestamp (milliseconds since Janaury 1, 1970), and a signature which is a HMAC SHA256 of the timestamp concatenated with the random token encrypted with your API Key found in the Webhooks tab. 

In order to decode it, please refer to this ruby on rails example:

```ruby
    token = params['signature']['random_token']
    timestamp = params['signature']['timestamp']
    signature = params['signature']['signature']

    signed_test_data = OpenSSL::HMAC.hexdigest("SHA256",
          ENV['CCPATollFree_WEBHOOK_PRIVATE_API_TOKEN'], "#{timestamp}#{token}")

    if signed_test_data == signature
        # the request is accpeted
    else
      raise "CCPATollfree.com did not authenticate"
    end
```

Follow best procedures to store and access your private API key. In this example, we stored it securely as an environment variable.

Another set of attacks on webhooks are replay attacks. An attacker may find that they can replay the full https encrypted webhook. Please check the timestamp at which it was signed. 

## Automatic Retry, and Manual Retry ##

We will automatically retry three times 30 minutes apart before stopping retrying. Then it will on you to retry the events manually by going into the your privacy manager and click the retry icon on the webhooks log. If you wish to retry all failed events, we will have an API for that. In the meantime please contact us if you have custom request as such.

The timestamp of the signature is recalculated, and the latest state of the data is fetched again, and then pushed to you. 


## Developing ##

We currently support 2 events:

```
privacy_request.received
privacy_request.updated
```

Those events will contain the latest state of the privacy request data. When a voicemail privacy request comes in, you may receive 2 privacy_request.updated after that - one when the voicemail recording is available, and another once the transcription is ready. Here is post example or privacy_request.received for a webform:

```javascript
{ event_name: "privacy_request.received",
  signature: {
    random_token: "b39a5c7ac85ec479f921cdfaae4b4eee",
    timestamp: "1584300477293",
    signature: "599a40fb2d25f995a40dc43f725a57be4e8df90d2574f75a498eed0d00bebaeb"
  },
  id: "72236cca-c0ee-4c43-8e10-d90737557a66",
  type: "WebForm",
  acknowledged: "false",
  extended: "false",
  completed: "false",
  deadline: "2020-04-28",
  created_at: "2020-03-14 18:44:40 UTC",
  updated_at: "2020-03-14 18:44:40 UTC",
  service_code: {
    code: "57",
    name: "tes",
    created_at: "2020-01-24 15:58:28 UTC",
    updated_at: "2020-01-31 21:15:24 UTC"
  },
  web_form_session: { 
    id: "560179c2-45f6-4168-b5cd-dd640e0c46b2",
    created_at: "2020-03-14 18:44:40 UTC",
    first_name: "Julian",
    last_name: "Salama",
    email: "admin@privacytollfree.com",
    requester_type: "Job Applicant",
    tell_me: "false",
    delete_me: "false",
    do_not_sell: "true",
    other: "",
    send_me: "false",
    remote_ip: "::1",
    state: "DE",
  }
}
```



```
{ event_name: "privacy_request.updated",
   signature: {
    random_token: "a021778c09cdae8897859340d3f0b32f"
    timestamp: "1584305606907"
    signature: "201de4703b7ce1eb33878fa29d77d8572ddeb2affd8e7530397d96c0f652eb90"
  }
  id: "abf78bbb-a152-4f09-90ad-5802f53721d7",
  type: "Voicemail",
  acknowledged: "false",
  extended: "false",
  completed: "true",
  deadline: "2019-11-05",
  created_at: "2019-09-21 16:31:11 UTC",
  updated_at: "2020-03-15 20:48:05 UTC",
  service_code: {
    code: "2"
    name: "test"
    created_at: "2019-09-21 16:31:11 UTC"
    updated_at: "2020-01-26 05:59:40 UTC"
  },
  call_session: {
    recording_status: ""
    mp3_encoded_bytes: ""
    created_at: "2019-09-21 16:31:11 UTC"
    ended_at: "2019-09-21 16:33:11 UTC"
    caller_name: "John Smith"
    caller_state: "CA"
    transcription_status: "completed"
    transcription_text: "Hello world!"
  }
}

```


As previously states both events will push the latest privacy request data. There is only two types of requests, WebForm and voicemail. The field call_session will be available for a voicemail privacy request, web_form_session will be available for a webform request.

## Here is a description of the state of each object ##

# Privacy Request

Key | Description
------------ | -------------
event_name | String, eg: "privacy_request.received"
signature | Object, with a random token, a timestamp in milliseconds, and a signature. Please refer to the security section to use this
id | UUID, the id as a string of the privacy request in our system.
type | String, e.g. "Voicemail"
acknowledged | Boolean, e.g: "false" represent if the privacy request was acknowledged.
complted | Boolean, e.g: "true" represent if the privacy request was completed.
extended | Boolean, e.g: "false" represent if the privacy request was extended.
deadline | Date, eg: "2019-11-05", calculated based on the extennded field, and created_at
created_at | Timestamp, the time at which we received this request
updated_at | Timestamp, the last time this object was updated.

# Service code #

Key | Description
------------ | -------------
code | String, your service code for this request.
name | String, your service code name for this request
created_at | Timestamp at which your created the service code
updated_at | Timestamp, the last time this object was updated.

# Call Session #

Key | Description
------------ | -------------
recording_status | String, "completed", when bytes are available, or "failed".
mp3_encoded_byes | String, a based64 encoded string representing the bytes of the mp3
created_at | Timestamp at we received the call for this request
ended_at | Timestamp when the caller hung up
caller_id | String, the phone number we received
caller_name | String CNAM Lookup of the caller name
caller_state | String e.g. "CA" state of the consumer derived from area code of the number
transcription_status | String, "completed" if we completed the transcription.
transcription_text | "Hello world!"


# Call Session #

Key | Description
------------ | -------------
id | String, UUID of the call session
recording_status | String, "completed", when bytes are available, or "failed".
mp3_encoded_byes | String, a based64 encoded string representing the bytes of the mp3
created_at | Timestamp at we received the call for this request
ended_at | Timestamp when the caller hung up
caller_id | String, the phone number we received
caller_name | String CNAM Lookup of the caller name
caller_state | String e.g. "CA" state of the consumer derived from area code of the number
transcription_status | String, "completed" if we completed the transcription, or "failed", or null if we the service code didn't enable transcription in your service code tab.
transcription_text | String "Hello world!"


# Web From Session #
Key | Description
------------ | -------------
id | UUID of the web form session
created_at | Timestamp at we received the webform for this request
first_name | String, first name of the requester
last_name | String, last name of the requester
email | String, email of the requester
state | String State of the requester reported on the webform eg: "DE"
requester_type | String type of requester. e.g.: "Consumer"
send_me | Boolean, eg: "true" User request that you send a copy of his information
tell_me | Boolean, eg: "false" User request that you tell him more about the information collected and why.
delete_me | Boolean, eg: "false" User request that you delete the information collected about him.
do_not_sell | Boolean, eg: "true" User Request that you stop selling his information.
other | String, a message from the user.
remote_ip:  | String, The IP address of the requester


## Edge Case ##

You have setup your webhook after you received a couple request, and you are receiving privacy_request.updated events without their corresponding privacy_request.created. You will have to create the privacy_requests now in your system. Don't forget you could be receiving multiple privacy_request.updated events, and therefore you will need a locking strategy. You may decide to lock on a parent object, or use an upsert if your database supports updating to the values of the latest timestamp signature. 




