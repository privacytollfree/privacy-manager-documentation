# CCPA Toll Free, Privacy Manager Documentation

### Get Started with [CCPA Toll Free](https://ccpatollfree.com) Webhooks, the value provider for toll free number for and privacy request manager.  ###

## Webhooks Introduction ##

Webhooks from your privacy manager at [ccpatollfree.com](https://ccpatollfree.com) allows you to retrieve data from your privacy requests in real time and securely. We only support webhook https URL. We support standard cypher suites and we believe it's your right to chose the right https cypher. Our webhooks are triggered based on events happening in our system. For instance, a privacy_request.received sends all data about the privacy request that's available at that time. Each event always include the latest state of the object you are about to receive - even under retry. We are not maintaining a list of changes, rather we order every events in the system and send all the data available. It's safe to use the signature timestamp as your means to order the object states. You should keep that signature timestamp to safeguard against older events. If you replay an older event, we will fetch the latest state of the object, and the signature timestamp will be the time now, so you are safe to update your record. Note that we are in the process of building a full API, and you may reach out to us if you wish to disclose your use-case. All http requests are multipart due to the nature of CCPA privacy request.

## Security ##

When it comes to webhook, you must validate that the data comes from us. The signature parameters in our request to your endpoint will contain three fields: a random token, a timestamp (milliseconds since Janaury 1, 1970), and a signature which is a HMAC SHA256 of the timestamp concatenated with the random token encrypted with your API Key found in the Webhooks tab in your dashboard. 

In order to decode it, please refer to this ruby on rails example:

```ruby
    token = params['signature']['random_token']
    timestamp = params['signature']['timestamp']
    signature = params['signature']['signature']

    signed_test_data = OpenSSL::HMAC.hexdigest("SHA256",
          ENV['CCPATollFree_WEBHOOK_PRIVATE_API_TOKEN'], "#{timestamp}#{token}")

    if signed_test_data == signature
        # the request is accepted
    else
      raise "CCPATollfree.com did not authenticate"
    end
```

Follow best procedures to store and access your private API key. In this example, we stored it securely in an environment variable.

Another set of attacks on webhooks are replay attacks. An attacker may find that they can replay the full https encrypted webhook. Please check the timestamp at which it was signed, and you may have add a 5 minute check and not allow those.

## Automatic Retry, and Manual Retry ##

We will automatically retry three times 30 minutes apart before stopping when a webhook fails. Then it will be on you to retry the events manually by going into the your privacy manager and click the retry icon on the webhooks log. If you wish to retry all failed events, we will have an API for that. In the meantime please contact us if you have custom requests as such.

The timestamp of the signature is recalculated at the time of the retry. The latest state of the data is fetched again, and then pushed to you. We have two separate status: recording_status, and transcription_status. Those two when "completed" will have mp3_encoded_bytes and transcription_text ready respectivaly. 


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

Here is an example of a privacy_request.updated for a voicemail request:


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
    created_at: "2019-09-21 16:31:11 UTC"
    ended_at: "2019-09-21 16:33:11 UTC"
    caller_id: "+15555555555"
    caller_name: "John Smith"
    caller_state: "CA"
    recording_status: ""
    mp3_encoded_bytes: ""
    transcription_status: "completed"
    transcription_text: "Hello world!"
  }
}

```


As previously stated both events will push the latest privacy request data. There is only two types of requests, WebForm and voicemail. Call_session will be available for a voicemail privacy request, web_form_session will be available for a webform request.

## Here is a description of the state of each object ##

# Privacy Request

Key | Description
------------ | -------------
event_name | String, eg: "privacy_request.received"
signature | Object, with a random token, a timestamp in milliseconds, and a signature. Please refer to the security section to use this
id | UUID, the id as a string of the privacy request in our system.
type | String, e.g. "Voicemail"
acknowledged | Boolean, e.g: "false" represent if the privacy request was acknowledged.
completed | Boolean, e.g: "true" represent if the privacy request was completed.
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
created_at | Timestamp at we received the call for this request
ended_at | Timestamp when the caller hung up
caller_id | String, the phone number we received
caller_name | String CNAM Lookup of the caller name
caller_state | String e.g. "CA" state of the consumer derived from area code of the number
recording_status | String, "completed", when bytes are available, or "failed".
mp3_encoded_byes | String, a based64 encoded string representing the bytes of the mp3
transcription_status | String, "completed" if we completed the transcription.
transcription_text | "Hello world!"


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


## Edge Case And Recommendations ##

You have setup your webhook after you received a couple request, and you are receiving privacy_request.updated events without their corresponding privacy_request.created. I recommend you correct those requests manually, or simply start recording the privacy requests into your system after the ones in transit are done. If you need us to send webhooks of all the previous privacy requests, let us know.

Sometimes, we can send a privacy_request.updated soon after it was received due to the mp3 being available, and the transcription being available. It could happen that it makes it in before the privacy_request.received. The simplest solution is to create the privacy request record in your system when receiving privacy_request.received. And update the fields based on the timestamp of the privacy_request.updated. If you do receive a privacy_request.updated before a privacy_request.received, you can just throw a specific error in your system, PrivacyRequestNotFound, and we will automatically retry that event in 30 minutes if we received anything else than a 200. It should not happen often, but if it does you may either have a locking strategy on a parent record like a service code record, use upsert based on latest timestamp, or write out those failed privacy_request.updated event and have a job that will retry them on your own system a few minutes later hoping you have received the privacy_request.received by then.

