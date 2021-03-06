Table of Contents
=================

   * [The Traveler](#the-traveler)
      * [Getting Started](#getting-started)
         * [Prerequisites](#prerequisites)
      * [OAuth](#oauth)
      * [Notices](#notices)
         * [Typical Response](#typical-response)
         * [Privacy](#privacy)
         * [Documentation](#documentation)
      * [Examples](#examples)
         * [Search for an Destiny Account on PSN](#search-for-an-destiny-account-on-psn)
         * [Get a character for an PSN Account](#get-a-character-for-an-psn-account)
      * [Progress](#progress)
      * [Built With](#built-with)
      * [Versioning](#versioning)
      * [Issues](#issues)
      * [License](#license)
      * [Acknowledgments](#acknowledgments)


# The Traveler

The Traveler is a small npm package which wraps around the Destiny 2 API. It uses Promises for a modern workflow in your application.

## Getting Started


```
npm install the-traveler
# or
yarn add the-traveler
```

### Prerequisites

To use this package you will need to obtain an API access key from the Bungie.net developer webiste. Please visit [https://www.bungie.net/en/Application](https://www.bungie.net/en/Application) to obtain your access token.

After obtaining your access token you are ready for using the Destiny 2 API in your next awesome project.

```
import Traveler from 'the-traveler';
import { ComponentType } from 'the-traveler/build/enums';

const traveler = new Traveler({
    apikey: 'pasteYourAPIkey',
    userAgent: 'yourUserAgent', //used to identify your request to the API
});
```

If you want to use this package inside a ES5 project you can import it like so:

```
var Traveler = require('the-traveler').default;
const Enums = require('the-traveler/build/enums')

const traveler = new Traveler({
    apikey: 'yourAPIkey',
    userAgent: 'yourUserAgent' //used to identify your request to the API
});

//Access the enums (example componentType profiles)
var profilesType = Enums.ComponentType.Profiles;

```

_If you want to show the URLs the API wrapper is requesting, just add `debug: true` to the configuration array when instantiate a `Traveler` object_
```
const traveler = new Traveler({
    apikey: 'pasteYourAPIkey',
    userAgent: 'yourUserAgent', //used to identify your request to the API
    debug: true 
});
```

## OAuth

If you want to use OAuth to also get access to endpoints which require authentications, provide the `Traveler` object with the needed OAuth `clientId` and if you are using a confidential client type the `clientSecret`.

```
import Traveler from 'the-traveler';

const traveler = new Traveler({
    apikey: 'pasteYourAPIkey',
    userAgent: 'yourUserAgent', //used to identify your request to the API
    oauthClientId: 'yourClientId',
    oauthClientSecret: 'yourClientSecret',
});
```


 Also ensure that you specify an `redirectURL` in your Application settings on [https://www.bungie.net/en/Application](https://www.bungie.net/en/Application)
After that you can generate the authentication URL which has to be visited by your users to approve your app and give it access. This follows the following schema: `https://www.bungie.net/en/OAuth/Authorize?client_id={yourClientID}&response_type=code`. 

```
const authUrl = traveler.traveler.generateOAuthURL(); // The URL your users have to visit to give your application access

```


If the users visit this site and approve your application they will be redirected to the `redirectURL` you specified, with a URL query parameter `code` appended: `https://www.example.com/?code=hereComesTheCode`

This is the code you need to get the OAuth Access token. Use it with the `getAccessToken()`

```
traveler.getAccessToken(hereComesTheCode).then(oauth => {
    // Provide your traveler object with the oauth object. This is later used for making authenticated calls
    traveler.oauth = oauth; 
}).catch(err => {
    console.log(err)
})
```

The oauth response schema depends on if you are using a `public` or `confidential`client type. With a `public` type the response does **not** contain an `refresh_token`. This means that a user has to authenticate everytime again after the OAuth access token has expired. Such an response looks like this:


_Response_:
```
{ access_token: '',
  token_type: 'Bearer',
  expires_in: 3600,
  membership_id: ''}
```

If you are using a `confidential` client type the response will contain an `refresh_token` which can be used to get a new `access_token` without requiring the user to approve your app again. Use this `refresh_token` to prevent getting errors because the `access_token` has expired.  In the following you can see such a response with the method to renew the token.

_Response_:
```
{ access_token: '',
  token_type: 'Bearer',
  expires_in: 3600,
  refresh_token: ',
  refresh_expires_in: 7776000,
  membership_id: '' }

``` 
_Use refresh_:
```
traveler.refreshToken(traveler.oauth.refresh_token).then(oauth => {
    // Provide your traveler object with the oauth object. This is later used for making authenticated calls
    traveler.oauth = oauth; 
}).catch(err => {
    console.log(err)
})
```

To wrap this up, the flow is the following:

* Provide `clientId` and `clientSecret (only for confidential)`
* Generate `authUrl` and redirect users to it
* Grab the `code` parameter from your `redirectURL`
* Use `code` to get the `oauth Object` and apply it to the `traveler object`
* **FOR Public** Reauthenticate your users after the access token has expired, so they have to visit the `authUrl`again
* **FOR Confidential** Use `traveler.oauth.refreshtoken` to renew the `accessToken`, without user interaction
* After the oauth object is set on the traveler object you can query the endpoints which require authentiation
* Keep in mind that it would be very useful to store the tokens for your users _**securely**_!

## Notices

There are some noteworthy information which could help to resolve some issues with the Destiny 2 API.

### Typical Response

The response object from the API is always constructed like the following snippet indicates. The `Response` will contain the actual request data, while `ErrorCode`, `ThrottleSeconds`, `ErrorStatus`, `Message` and `MessageData` will hold additional data about the sucess of our request. 

```
{ Response: Array or Object,
  ErrorCode: 1,
  ThrottleSeconds: 0,
  ErrorStatus: 'Success',
  Message: 'Ok',
  MessageData: {} }
```
### Privacy

Some information in the Destiny API is privacy protected. If the user set the pricacy settings it is not possible to obtain specific information through the API. The different pricacy indicators are the following:

* None: 0
* Public: 1
* Private: 2

### Documentation

* The documentation for this package can be found at [The Traveler documentation](https://alexanderwe.github.io/the-traveler/)
* A fully complete documentation about the different endpoints and methods can be found at the [official Destiny 2 API site](https://bungie-net.github.io/multi/operation_get_Destiny2-GetDestinyManifest.html#operation_get_Destiny2-GetDestinyManifest)


## Examples

### Search for an Destiny Account on PSN

_Query:_
```
traveler
    .searchDestinyPlayer('2', 'playername')
    .then(player => {
        console.log(player);
    }).catch(err => {
        //do something with the error
    })
```

_Response_
```
{ Response:
   [ { iconPath: '/img/theme/destiny/icons/icon_psn.png',
       membershipType: 2,
       membershipId: '4611686018433838874',
       displayName: 'Playername' } ],
  ErrorCode: 1,
  ThrottleSeconds: 0,
  ErrorStatus: 'Success',
  Message: 'Ok',
```

### Get a character for an PSN Account

Here all character specific components are queried. You can either use normal strings or use the integrated enums for a better naming.

_Query:_
```
import Traveler from 'the-traveler';
import {ComponentType} from 'the-traveler/build/enums' 

const traveler = new Traveler({
    apikey: 'pasteYourAPIkey',
    userAgent: 'yourUserAgent' //used to identify your request to the API
});

traveler.getCharacter('2', '4611686018452033461', '2305843009265042115', { components: ['200', '201', '202', '203', '204', '205'] }).then(result => {
    console.log(result);
}).catch(err => {
    console.log(err);
});

// OR

traveler.getCharacter('2', '4611686018452033461', '2305843009265042115', {
    components:
    [
        ComponentType.Characters,
        ComponentType.CharacterInventories,
        ComponentType.CharacterProgressions,
        ComponentType.CharacterRenderData,
        ComponentType.CharacterActivities,
        ComponentType.CharacterEquipment
    ]
}).then(result => {
    console.log(result);
}).catch(err => {
    console.log(err);
});

```
_Response (First level):_
```
{ Response:
   { inventory: { privacy: 2 },
     character: { data: [Object], privacy: 1 },
     progressions: { privacy: 2 },
     renderData: { data: [Object], privacy: 1 },
     activities: { privacy: 2 },
     equipment: { data: [Object], privacy: 1 },
     itemComponents: {} },
  ErrorCode: 1,
  ThrottleSeconds: 0,
  ErrorStatus: 'Success',
  Message: 'Ok',
  MessageData: {} }
```


## Progress

Please visit the [official documentation for the API](https://bungie-net.github.io/multi/operation_get_Destiny2-GetDestinyManifest.html#operation_get_Destiny2-GetDestinyManifest) to check if the endpoints are working or if they are still in preview. If you find endpoints in preview, please bear in mind that errors can occur quite often. If the endpoints get finalized also this package will adopt changes and test the functionalities.

| Endpoint                                  | Implemented      | Unlocked in API       |
| ----------------------------------------- | ---------------- | --------------------- |
| Destiny2.GetDestinyManifest               | ![alt text][yes] | ![alt text][unlocked] |
| Destiny2.SearchDestinyPlayer              | ![alt text][yes] | ![alt text][unlocked] |
| Destiny2.GetProfile                       | ![alt text][yes] | ![alt text][unlocked] |
| Destiny2.GetCharacter                     | ![alt text][yes] | ![alt text][unlocked] |
| Destiny2.GetClanWeeklyRewardState         | ![alt text][yes] | ![alt text][unlocked] |
| Destiny2.GetItem                          | ![alt text][yes] | ![alt text][unlocked] |
| Destiny2.GetVendors                       | ![alt text][yes] | ![alt text][preview]  |
| Destiny2.GetVendor                        | ![alt text][yes] | ![alt text][preview]  |
| Destiny2.TransferItem                     | ![alt text][no]  | ![alt text][unlocked] |
| Destiny2.EquipItem                        | ![alt text][no]  | ![alt text][unlocked] |
| Destiny2.EquipItems                       | ![alt text][no]  | ![alt text][unlocked] |
| Destiny2.SetItemLockState                 | ![alt text][no]  | ![alt text][unlocked] |
| Destiny2.InsertSocketPlug                 | ![alt text][no]  | ![alt text][preview]  |
| Destiny2.ActivateTalentNode               | ![alt text][no]  | ![alt text][preview]  |
| Destiny2.GetPostGameCarnageReport         | ![alt text][yes] | ![alt text][unlocked] |
| Destiny2.GetHistoricalStatsDefinition     | ![alt text][yes] | ![alt text][unlocked] |
| Destiny2.GetClanLeaderboards              | ![alt text][yes] | ![alt text][preview]  |
| Destiny2.GetClanAggregateStats            | ![alt text][yes] | ![alt text][preview]  |
| Destiny2.GetLeaderboards                  | ![alt text][yes] | ![alt text][preview]  |
| Destiny2.GetLeaderboardsForCharacter      | ![alt text][yes] | ![alt text][preview]  |
| Destiny2.SearchDestinyEntities            | ![alt text][yes] | ![alt text][preview]  |
| Destiny2.GetHistoricalStats               | ![alt text][yes] | ![alt text][preview]  |
| Destiny2.GetHistoricalStatsForAccount     | ![alt text][yes] | ![alt text][preview]  |
| Destiny2.GetActivityHistory               | ![alt text][yes] | ![alt text][preview]  |
| Destiny2.GetUniqueWeaponHistory           | ![alt text][yes] | ![alt text][preview]  |
| Destiny2.GetDestinyAggregateActivityStats | ![alt text][yes] | ![alt text][preview]  |
| Destiny2.GetPublicMilestoneContent        | ![alt text][yes] | ![alt text][unlocked] |
| Destiny2.GetPublicMilestones              | ![alt text][yes] | ![alt text][unlocked] |

## Built With

* [request-promise-native](https://github.com/request/request-promise-native) - Request package
* [Typescript](https://github.com/Microsoft/TypeScript) - Programming Language

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/alexanderwe/the-traveler/tags). 


## Issues

Do you have any issues or recommendations for this package ? Feel free to open an issue in the [isse section](https://github.com/alexanderwe/the-traveler/issues) :) 


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

**the-traveler package isn't endorsed by Bunge and doesn't reflect the views or opinions of Bungies or anyone officially involved in producing or managing Destiny 2. Destiny 2 and Bungie are trademarks or registered trademarks of Bungie, Inc. Destiny 2 © Bungie.**


[yes]:  https://img.shields.io/badge/implemented-yes-green.svg "Implemented:yes"
[no]:  https://img.shields.io/badge/implemented-no-red.svg "Implemented:no"
[preview]: https://img.shields.io/badge/-preview-blue.svg "Preview"
[unlocked]: https://img.shields.io/badge/-unlocked-green.svg "Unlocked"