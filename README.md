# Reactive Location API Library for Android

[![Build Status](https://travis-ci.org/patloew/RxLocation.svg?branch=master)](https://travis-ci.org/patloew/RxLocation) [![API](https://img.shields.io/badge/API-9%2B-brightgreen.svg?style=flat)](https://android-arsenal.com/api?level=9)

This library wraps the Location APIs in [RxJava 2](https://github.com/ReactiveX/RxJava/tree/2.x) Observables, Singles and Maybes. No more managing GoogleApiClients! Also, the resolution of the location settings check is optionally handled by the lib.

For [RxJava 1](https://github.com/ReactiveX/RxJava/tree/1.x), please take a look at the [Android-ReactiveLocation](https://github.com/mcharmas/Android-ReactiveLocation) library by Michał Charmas.

# Usage

Create an RxLocation instance once, preferably in your Application's `onCreate()` or by using a dependency injection framework. The RxLocation class is very similar to the classes provided by the Location APIs. Instead of `LocationServices.FusedLocationApi.getLastLocation(apiClient)` you can use `rxLocation.location().lastLocation()`. Make sure to have the Location and Activity Recognition permissions from Marshmallow on, if they are needed by your API requests.

Example:

```java
// Create one instance and share it
RxLocation rxLocation = new RxLocation(context);

LocationRequest locationRequest = LocationRequest.create()
                .setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY)
                .setInterval(5000);

rxLocation.location().updates(locationRequest)
		.flatMap(location -> rxLocation.geocoding().fromLocation(location).toObservable())
		.subscribe(address -> {
			/* do something */
		});
```

The following APIs are wrapped by this library:

* `ActivityRecognition.ActivityRecognitionApi` via `rxLocation.activity()`
* `LocationServices.FusedLocationApi` via `rxLocation.location()`
* `LocationServices.GeofencingApi` via `rxLocation.geofencing()`
* `LocationServices.SettingsApi` via `rxLocation.settings()`
* `Geocoder` via `rxLocation.geocoding()`

An optional global default timeout for all Location API requests made through the library can be set via `rxLocation.setDefaultTimeout(...)`. In addition, timeouts can be set when creating a new Observable by providing timeout parameters, e.g. `rxLocation.location().updates(locationRequest, 15, TimeUnit.SECONDS)`. These parameters override the default timeout. When a timeout occurs, a StatusException is provided via `onError()`. The RxJava timeout operators can be used instead, but these do not cancel the Location API request immediately.

You can also obtain a `Single<GoogleApiClient>`, which connects on subscribe and disconnects on unsubscribe via `GoogleAPIClientSingle.create(...)`.

The following Exceptions are thrown in the lib and provided via `onError()`:

* `StatusException`: When the call to the Location APIs was not successful or timed out
* `GoogleAPIConnectionException`: When connecting to the GoogleAPIClient was not successful.
* `GoogleAPIConnectionSuspendedException`: When the GoogleApiClient connection was suspended.
* `SecurityException`: When you try to call an API without proper permissions.
* `LocationSettingsNotSatisfiedException`: When you use `rxLocation.settings().checkAndHandleResolutionCompletable(...)` and the location settings were not satisfied, even after handling the resolution.

# Sample

A basic sample app is available in the `sample` project.

# Testing

When unit testing your app's classes, RxLocation behavior can be mocked easily. See the `MainPresenterTest` in the `sample` project for an example test.

# Credits

The code for managing the GoogleApiClient was adapted from the [Android-ReactiveLocation](https://github.com/mcharmas/Android-ReactiveLocation) library by Michał Charmas, which is licensed under the Apache License, Version 2.0.

# License

	Copyright 2016 Patrick Löwenstein

	Licensed under the Apache License, Version 2.0 (the "License");
	you may not use this file except in compliance with the License.
	You may obtain a copy of the License at

	    http://www.apache.org/licenses/LICENSE-2.0

	Unless required by applicable law or agreed to in writing, software
	distributed under the License is distributed on an "AS IS" BASIS,
	WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
	See the License for the specific language governing permissions and
	limitations under the License.