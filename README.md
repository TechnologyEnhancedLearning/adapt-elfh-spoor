# adapt-elfh-spoor
<a id="top" style="display:none"></a>

The **e-LfH Spoor** *extension* is for use with the the [Adapt framework](https://github.com/adaptlearning/adapt_framework).  It is based on the [adapt-contrib-spoor](https://github.com/adaptlearning/adapt-contrib-spoor) extension, which adds SCORM 1.2 tracking functionality to Adapt courses.  The e-LfH Spoor plugin adds **AICC HACP (HTTP-based AICC/CMI protocol)** tracking functionality, on top of existing SCORM tracking made available inside the adapt-contrib-spoor extension.


Please be aware that this extension has so far been tested and verified to work in the following LMS's.  If you encounter any issues with other LMS's please report them:
- e-Learning for Healthcare's Hub
- Moodle version 3.0 release candidate 4

AICC HACP allows organisations to run courses in their LMS applications, which are sitting on another organisation's domain, usually inside some sort of content repository, without causing [cross site scriping](https://en.wikipedia.org/wiki/Cross-site_scripting) errors in the user's web browser.  Running courses in this way is not possible using SCORM, because a web browser will throw a cross site scripting error and prevent the course from running for security reasons.  Please note that AICC HACP will only work if an application exists to relay the HTTP requests posts to and from the LMS.  This **relay** application often exists as part of an organisation's content repository.  This extension completes one part of the AICC HACP implementation but a relay application and an LMS capable of launching external AICC content, is required to make the entire process work.

**e-LfH Spoor** makes use of the [pipwerks SCORM API Wrapper](https://github.com/pipwerks/scorm-api-wrapper/).  Built on top of this wrapper, this extension has been designed so the additional AICC functionality is added unobstrusively.  Source code in the files aiccAPI.js and aiccLMS.js contain the additional tracking functionality and wrapper.js has been changed slightly to detect whether a request to run the course is made via HACP AICC or SCORM.  This means an Adapt course can be created which can switch between either AICC or SCORM using this single extension, without the need to create two versions of the same course, e.g. one version of the course containing the SCORM extension and another version containing an AICC extension.  Therefore removing the need to update two courses if the content is changed.

An Adapt course built to include this extension will run in one of three possible *modes*:

- if the course is launched outside of an LMS as a stand-alone website, the extension will not detect an AICC or SCORM LMS and the course will run without any tracking enabled.
- if the extension detects an AICC identifier and url in the querystring, then the course will run with AICC tracking enabled.
- if the extension detects a SCORM API in a parent frame, window or opener window, then the course will run with SCORM tracking enabled.

It's important to note that this extension doesn't attempt to implement the entire [AICC specification](https://github.com/ADL-AICC/AICC-Document-Archive/), but more specifically section *6.0 Communicating via HTTP (The HACP Binding)* in the document *CMI Guidelines for Interoperability AICC* ([cmi001v4.pdf](https://github.com/ADL-AICC/AICC-Document-Archive/releases/tag/cmi001v4)).  The extension adds the ability for an [LMS](https://en.wikipedia.org/wiki/Learning_management_system) that has HACP AICC functionality (such as [Moodle](https://moodle.org/), or [Kallidus](https://www.kallidus.com/)), to store values returned from a course stored on a domain other than the domain the LMS is running from.

For example, the course could be sitting on a domain that is acting as a content repository, e.g. *http://my-company/courses/course1*, and your client may want to run the course from their own Moodle LMS instance on their domain *http://the-client-LMS/*.  The administrator of the client's LMS would create a course in the LMS and configure it to point to the Adapt courses (containing the e-LfH Spoor extension) sitting on the other domain.

## Installation

This plugin should be used instead of the core Spoor plugin (adapt-contrib-spoor), when both SCORM and AICC tracking might be required.  To use this extension, uninstall adapt-contrib-spoor and install adapt-elfh-spoor.

Within the Adapt authoring tool you can uninstall adapt-contrib-spoor and install adapt-elfh-spoor using the [Plug-in Manager](https://github.com/adaptlearning/adapt_authoring/wiki/Plugin-Manager).

If you are creating your course using the Adapt framework directly and outside of the authoring tool, you can use uninstall and install the extensions using the [Adapt CLI](https://github.com/adaptlearning/adapt-cli).  Run the following from the command line:

1. `adapt uninstall adapt-contrib-spoor`
2. `adapt install adapt-elfh-spoor`

### Configure *config.json*
**NOTE:**  as of Adapt/Spoor v3 you will first need to configure the settings in the **\_completionCriteria** object in config.json to specify whether you want course completion to be based on content completion, assessment completion, or both. (In earlier versions of Spoor these settings were part of the spoor configuration - but were moved to the core of Adapt so that they could be used by other tracking extensions such as xAPI.)

The attributes listed below are used in *config.json* to configure **Spoor**, and are properly formatted as JSON in [*example.json*](https://github.com/adaptlearning/adapt-contrib-spoor/blob/master/example.json). Visit the [**Spoor** wiki](https://github.com/adaptlearning/adapt-contrib-spoor/wiki) for more information about how they appear in the [authoring tool](https://github.com/adaptlearning/adapt_authoring/wiki).


## Limitations

Supports SCORM 1.2, but in AICC mode only the following fields from the specification are available and are tracked:

* Core.StudentId
* Core.StudentName
* Core.LessonLocation
* Core.LessonStatus
* Core.Exit
* Core.Entry
* Core.Score
* Core.SessionTime
* Core.TotalTime
* SuspendData

<div float align=right><a href="#top">Back to Top</a></div>

----------------------------
<a href="https://community.adaptlearning.org/" target="_blank"><img alt="@e-LfH" class="TableObject-item avatar" height="100" itemprop="image" src="https://avatars2.githubusercontent.com/u/30687181?v=4&amp;s=200" align="right"/></a>
**Version number:**  see bower.json
**Framework versions:** 3.0.0+
**Author / maintainer:** e-Learning For Healthcare with [contributors](https://github.com/e-LfH/adapt-elfh-spoor/graphs/contributors)
**Accessibility support:** n/a
**RTL support:** n/a
**Cross-platform coverage:** Chrome, Chrome for Android, Firefox (latest version), Edge, IE11, Safari 10+11 for macOS+iOS



## Attributes

### \_spoor (object):

The `_spoor` object contains the setting `_isEnabled` and the `_tracking`, `_reporting` and `_advancedSettings` objects.

#### \_isEnabled (boolean):
Enables/disables this extension. If set to `true` (the default value), the plugin will try to connect to a SCORM conformant LMS when the course is launched via *index_lms.html*. If one is not available, a 'Could not connect to LMS' error message will be displayed. This error can be avoided during course development either by setting this to `false` or - more easily - by launching the course via *index.html*. This latter technique is also useful if you are developing a course that could be run either from an LMS or a regular web server.

#### \_tracking (object):

This object defines what kinds of data to record to the LMS. It consists of the following settings:

##### \_shouldSubmitScore (boolean):
Determines whether the assessment score will be reported to the LMS. Note that SCORM only supports one score per SCO, so if you have multiple assessments within your course, one aggregated score will be recorded. Acceptable values are `true` or `false`. The default is `false`.

##### \_shouldStoreResponses (boolean):
Determines whether the user's responses to questions should be persisted across sessions (by storing them in `cmi.suspend_data`) or not. Acceptable values are `true` or `false`. The default is `true`. Note that if you set this to `true`, the user will not be able to attempt questions within the course again unless some mechanism for resetting them is made available (for example, see `_isResetOnRevisit` in [adapt-contrib-assessment](https://github.com/adaptlearning/adapt-contrib-assessment)).

##### \_shouldStoreAttempts (boolean):
Determines whether the history of the user's responses to questions should be persisted across sessions (by storing them in `cmi.suspend_data`) or not. Acceptable values are `true` or `false`. The default is `false`.

##### \_shouldRecordInteractions (boolean):
Determines whether the user's responses to questions should be tracked to  the `cmi.interactions` fields of the SCORM data model or not. Acceptable values are `true` or `false`. The default is `true`. Note that not all SCORM 1.2 conformant Learning Management Systems support `cmi.interactions`. The code will attempt to detect whether support is implemented or not and, if not, will fail gracefully. Occasionally the code is unable to detect when `cmi.interactions` are not supported, in those (rare) instances you can switch off interaction tracking using this property so as to avoid 'not supported' errors. You can also switch off interaction tracking for any individual question using the `_recordInteraction` property of question components. All core question components support recording of interactions, community components will not necessarily do so.

##### \_shouldCompress (boolean):
Allow variable LZMA compress on component state data. The default is `false`.

#### \_reporting (object):
This object defines what status to report back to the LMS. It consists of the following settings:

##### \_onTrackingCriteriaMet (string):
Specifies the status that is reported to the LMS when the tracking criteria (as defined in the `_completionCriteria` object in config.json) are met. Acceptable values are: `"completed"`, `"passed"`, `"failed"`, and `"incomplete"`. If you are tracking a course by assessment, you would typically set this to `"passed"`. Otherwise, `"completed"` is the usual value.

##### \_onAssessmentFailure (string):
Specifies the status that is reported to the LMS when the assessment is failed. Acceptable values are `"failed"` and `"incomplete"`. Some Learning Management Systems will prevent the user from making further attempts at the course after status has been set to `"failed"`. Therefore, it is common to set this to `"incomplete"` to allow the user more attempts to pass an assessment.

##### \_resetStatusOnLanguageChange (boolean):
If set to `true` the status of the course is set to "incomplete" when the languge is changed using the [adapt-contrib-languagePicker plugin](https://github.com/adaptlearning/adapt-contrib-languagepicker). Acceptable values are `true` or `false`. The default is `false`.

#### \_advancedSettings (object):

The advanced settings objects contains the following settings. Note that you only need to include advanced settings if you want to change any of the following settings from their default values - and you only need to include those settings you want to change.

##### \_scormVersion (string):
This property defines what version of SCORM is targeted. Only SCORM 1.2 and SCORM 2004 4th Edition files are included. To use a different SCORM 2004 Edition, replace the [*scorm/2004*](https://github.com/adaptlearning/adapt-contrib-spoor/blob/master/scorm/2004) files accordingly - examples can be found at [scorm.com](http://scorm.com/scorm-explained/technical-scorm/content-packaging/xml-schema-definition-files/). Acceptable values are `"1.2"` or `"2004"`. The default is `"1.2"`.

##### \_showDebugWindow (boolean):
If set to `true`, a pop-up window will be shown on course launch that gives detailed information about what SCORM calls are being made. This can be very useful for debugging SCORM issues. Note that this pop-up window will appear automatically if the SCORM code encounters an error, even if this is set to `false`. You can also hold down the keys <kbd>d</kbd>+<kbd>e</kbd>+<kbd>v</kbd> to force the popup window to open. The default is `false`.

##### \_suppressErrors (boolean):
If set to `true`, an alert dialog will NOT be shown when a SCORM error occurs. Errors will still be logged but the user will not be informed that a problem has occurred. Note that setting `_showDebugWindow` to `true` will still cause the debug popup window to be shown on course launch, this setting merely suppresses the alert dialog that would normally be shown when a SCORM error occurs. *This setting should be used with extreme caution as, if enabled, users will not be told about any LMS connectivity issues or other SCORM tracking problems.*

##### \_commitOnStatusChange (boolean):
Determines whether a "commit" call should be made automatically every time the SCORM *lesson_status* is changed. The default is `true`.

##### \_commitOnAnyChange (boolean):
Determines whether a "commit" call should be made automatically _every time_ any SCORM value is changed. The default is `false`. Setting `_commitOnAnyChange` to `true` will disable 'timed commits'. **Note:** enabling this setting will make the course generate a lot more client-server traffic so you should only enable it if you are sure it is needed and, as it may have a detrimental impact on server performance, after careful load-testing. An alternative might be to first try setting a lower value for `_timedCommitFrequency`.

##### \_timedCommitFrequency (number):
Specifies the frequency - in minutes - at which a "commit" call will be made. Set this value to `0` to disable automatic commits. The default is `10`.

##### \_maxCommitRetries (number):
If a "commit" call fails, this setting specifies how many more times the "commit" call will be attempted before giving up and throwing an error. The default is `5`.

##### \_commitRetryDelay (number):
Specifies the interval in milliseconds between commit retries. The default is `2000`.

##### \_commitOnVisibilityChangeHidden (boolean):
Determines whether or not a "commit" call should be made when the [visibilityState](https://developer.mozilla.org/en-US/docs/Web/API/Document/visibilityState) of the course page changes to `"hidden"`. This functionality helps to ensure that tracking data is saved whenever the user switches to another tab or minimises the browser window - and is only available in [browsers that support the Page Visibility API](http://caniuse.com/#search=page%20visibility). The default is `true`.

##### \_manifestIdentifier (string):
Used to set the `identifier` attribute of the `<manifest>` node in imsmanifest.xml - should you want to set it to something other than the default value of `"adapt_manifest"`. Strictly speaking, this value is meant to be unique for each SCO on the LMS; in practice, few LMSes require or enforce this.

##### \_exitStateIfIncomplete (string):
Determines the 'exit state' (`cmi.core.exit` in SCORM 1.2, `cmi.exit` in SCORM 2004) to set if the course hasn't been completed. The default behaviour will cause the exit state to be set to an empty string for SCORM 1.2 courses, or `"suspend"` for SCORM 2004 courses. The default behaviour should be left in place unless you are confident you know what you are doing!

##### \_exitStateIfComplete (string):
Determines the 'exit state' (`cmi.core.exit` in SCORM 1.2, `cmi.exit` in SCORM 2004) to set when the course has been completed. The default behaviour will cause the exit state to be set to an empty string for SCORM 1.2 courses, or `"normal"` for SCORM 2004 courses. The default behaviour should be left in place unless you are confident you know what you are doing! Note: if you are using SCORM 2004, you can set this to `"suspend"` to prevent the LMS from clearing all progress tracking when a previously-completed course is re-launched by the learner.

##### \_setCompletedWhenFailed (boolean):
Determines whether the `cmi.completion_status` is set to "completed" if the assessment is "failed". Only valid for SCORM 2004, where the logic for completion and success is separate. The default is `true`.

##### \_connectionTest (object):
The settings used to configure the connection test when committing data to the LMS. The LMS API usually returns true for each data transmission regardless of the ability to persist the data. Contains the following attributes:

 * **\_isEnabled** (boolean): Determines whether the connection should be tested. The default is `true`.

 * **\_testOnSetValue** (boolean): Determines whether the connection should be tested for each call to set data on the LMS. The default is `true`.

   * **_silentRetryLimit** (number): The limit for silent retry attempts to establish a connection before raising an error. The default is `2`.

   * **_silentRetryDelay** (number): The interval in milliseconds between silent connection retries. The default is `1000`.

#### \_showCookieLmsResetButton (boolean):
Determines whether a reset button will be available to relaunch the course and optionally clear tracking data (scorm_test_harness.html only). The default is `false`.

#### \_shouldPersistCookieLMSData (boolean):
Determines whether to persist the cookie data over browser sessions (scorm_test_harness.html only). The default is `true`.

<div float align=right><a href="#top">Back to Top</a></div>

## Notes

### Running a course without tracking while Spoor is installed
- Use *index.html* instead of *index_lms.html*.
*OR*
- Set `"_isEnabled": false` in *config.json*.

### Client Local Storage / Fake LMS / Adapt LMS Behaviour Testing
When **Spoor** is installed, *scorm_test_harness.html* can be used instead of *index.html* to allow the browser to store LMS states inside a browser cookie. This allows developers to test LMS-specific behaviour outside of an LMS environment. If you run the command `grunt server-scorm`, this will start a local server and run the course using *scorm_test_harness.html* for you.

Note that due to the data storage limitations of browser cookies, there is less storage space available than an LMS would provide. As of [v2.1.1](https://github.com/adaptlearning/adapt-contrib-spoor/releases/tag/v2.1.1), a browser alert will be displayed if the code detects that the cookie storage limit has been exceeded.

~~In particular having `_shouldRecordInteractions` enabled can cause a lot of data to be written to the cookie, using up the available storage more quickly - it is advised that you disable this setting when testing via *scorm_test_harness.html*.~~ As of [v3.0.0](https://github.com/adaptlearning/adapt-contrib-spoor/releases/tag/v3.0.0) this is no longer an issue - 'interaction data' is no longer saved to the cookie. As cmi.interactions are 'write only' in the SCORM spec, there was no reason to be doing this as the data would never be used.

As of [v3.7.0](https://github.com/adaptlearning/adapt-contrib-spoor/releases/tag/v3.7.0), you can make the cookie 'persistent' if you want to be able to have the cookie persist for longer than the browser's 'session'. For example, you might want to make basic tracking & bookmarking functionality available to learners when the course is being run from a regular web server (rather than an LMS or LRS). Just be aware that this isn't officially supported by the Adapt Core Team, so if you want to use this you do so at your own risk! Please see [the comments in scorm_test_harness.html](https://github.com/adaptlearning/adapt-contrib-spoor/blob/ecf5c16ca022345e69b08f02a523eb773f24ba07/required/scorm_test_harness.html#L24-L29) for details on how to make the cookie 'persistent'.

### SCORM Error Messages
As of [v3.6.0](https://github.com/adaptlearning/adapt-contrib-spoor/releases/tag/v3.6.0) it's possible to amend and/or translate the error messages that are shown by this extension whenever an LMS error is encountered. See [*example.json*](https://github.com/adaptlearning/adapt-contrib-spoor/blob/master/example.json) for the data that needs to be added to course/_lang_/course.json

Note that you only need to include those you want to amend/translate.

These error messages can also be amended via the Adapt Authoring Tool - but must be supplied in JSON format. For example, if you wanted to translate the 'could not connect to LMS' error into French, you would added the following into the 'Error messages' field under Project settings > Extensions > Spoor (SCORM):
```json
"title": "Une erreur s'est produite",
"CLIENT_COULD_NOT_CONNECT": "La connexion avec la plate-forme de formation n'a pas pu être établie."
```

### Print completion information from LMS data
If you have a course where learners are reporting completion problems, it can often be useful to check the stored suspend data to see if they are simply missing something out. As the relevant part of the suspend data is no longer in 'human-readable' format, Spoor v3.8.0 includes a `printCompletionInformation` function that translates this into into a more readable string of 1s and 0s which you can then match to the course's 'tracking ids' to see which bits of the course the learner hasn't completed.

To do this, run any course that uses Spoor v3.8.0 (or better) and execute the following via the browser console. Naturally, you need to replace the `suspendData` shown below with the one from the course you're trying to debug...
```js
var suspendData  = '{"lang":"en","a11y":false,"captions":"en","c":"hAA","q":"oAPQ4XADAcATDAHC4EYDgCYYA4XEDAcATDAHC4kYDgCYYA4XIDAcATDAHC5gbDgCYYbjhdAMBwBAMAcLqBgOAIBgDhdgMBwBAMAcLuBgOFwBABgDhdyMBwBAMAcLQgDAcLgCADAHC0JAwHC4AgAwBwtCSMBwBAMAcLQoDAcLgCADAHC0LAwHC4AgAwBwtDAMBwuAIAMAcLQwjAcAQDAHC0NAwHC4AgAwBwtDSMBwBAMAcLQ4DAcLgCADAHC0OIwHAEAwBwtDwMBwuAIAMAcLQ8jAcAQDAHC0QAwHC4AgAwBwtECMBwBAMAcLREDAcLgCADAHC0RIwHAEAwBwtEgMBwBAMAcLRMDAcAQDAA"}';
require('core/js/adapt').spoor.statefulSession.printCompletionInformation(suspendData);
```
That will output something like the following:
```console
INFO: course._isComplete: false, course._isAssessmentPassed: false, block completion: 11110000000000000000
```
Which, in the above example, indicates that the learner only completed the blocks with trackingIds 0, 1, 2, & 3.
## Limitations

Currently (officially) only supports SCORM 1.2

----------------------------
**Version number:**  3.9.0   <a href="https://community.adaptlearning.org/" target="_blank"><img src="https://github.com/adaptlearning/documentation/blob/master/04_wiki_assets/plug-ins/images/adapt-logo-mrgn-lft.jpg" alt="adapt learning logo" align="right"></a>
**Framework versions:** 5.5+
**Author / maintainer:** Adapt Core Team with [contributors](https://github.com/adaptlearning/adapt-contrib-spoor/graphs/contributors)
**Accessibility support:** n/a
**RTL support:** n/a
**Cross-platform coverage:** Chrome, Chrome for Android, Firefox (ESR + latest version), Edge, IE11, Safari 14 for macOS/iOS/iPadOS, Opera

