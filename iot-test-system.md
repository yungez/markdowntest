# IOT Test Automation
## Revision History
|Date  |Version|By|Reviewers|Changes/Comments|
| ---- | ---- | ---- | ---- | ---- |
|2016-08-30|0.1|yaozheng||Initial Draft|
|2016-10-26|1.0|yaozheng||First version|

This document is about IoT test automation setup

## Scenarios
- **P1** Supported physical machine and environment (Windows/Linux/Mac) will be setup for end-to-end test run.
- **P1** The physical machines will be available in VSO to test build definitions.
- **P1** Selected core test case will be automatically triggered by code commit and merge.
- **P1** Test driver system will setup to run the different type to test cases:
    - **P1** Developer can run unit test and linter check in local environment
    - **P1** End-to-end test case can be run on the iot lab system
- **P1** Detailed test run report will be sent automatically to the related part.
- **P1** A wrapper lib for the iot lab system will be provided for e2e test cases.
- **P2** A portal to list the available hardware(PC, boards).

### Non Scope
This document is about test automation setup and run. For concrete unit test and e2e test cases, it's up to the feature author to write the test cases. And test automation will appropriately run the test case.

Currently there are several example test cases available for reference in the VSIoT/test/e2e/iot_samples folder.

## Integration with CI
- Test build definitions targeting each physical machines will be added.
- Test will be automatically triggered by commits to develop/master/stable branches.

## Run Test
- We will leverage the node.js ecosystem to setup the test environment since we have already using node.js & gulp.
- With node.js & gulp, our test case will be run cross-platform in the beginning.
- For cross language(C, C#, Python if any) test case, we can use node.js process to invoke the test cases.

Here is an example script for testing:
```
// Install the required components on the test environment:
$ npm install

// Do javascript coding styling check:
$ gulp eslint

// Run devdisco test for USB. The USB devices to be verified are listed in config file.
$ gulp devdisco --spec=usb

// Run devdisco test for Ethernet. The Ethernet devices to be verified are listed in config file.
$ gulp devdisco --spec=eth

// Run devdisco test for Wi-Fi. The Wi-Fi devices to be verified are listed in config file.
$ gulp devdisco --spec=wifi

// Run devdisco test for USB, Wi-Fi and Ethernet.
$ gulp devdisco

// Run unittest
$ gulp unittest

// Run E2E test
$ gulp e2e

// Also, we can run specific test case or test scenario for diagnostic purpose:
$ gulp e2e --spec=raspberrypi

// For testing against specific branch
$ gulp e2e --spec=huzzah --branch=my_branch

// For testing using ssh instead of https to clone repo in the lab.
$ gulp e2e --spec=huzzah --branch=my_branch --git=ssh

// For full testing run including dynamic creation and destroy of Azure resource group, IoT Hub and deployment.
 $ gulp e2e --spec=huzzah --branch=my_branch --git=ssh --fullTestRun

```

- Local dev experience: It's supported to run tests against local sample/devdisco code or local gulp-common repo. Scenario is that, I have my own sample code repo, and some private code change in it. Before checkin, I'd like to run e2e test against it to verify my change, instead of against public sample code repo at Github.

Here is example script for testing:
```
$ gulp e2e --spec=thingdev --localrepo=../../../iot_sample_repos/iot-hub-c-thingdev-azure-blink --localgulpcommon=/d/vsiot1/gulp-common
--localrepo option is used to specify local private sample code repo location.
--localgulpcommon option is used to specify local gulp-common repo location.

$ gulp devdisco --usb --localrepo="/d/device-discovery-cli"
--localrepo option is used to specify local private sample code repo location.
```

## Gated check-in

For develop/master branch, the related test(linter/e2e) will be automatically triggered for each commit.
    - Commits/PR are happening on the github
    - The test build definition for the related github repo will be triggered on the VSTS side by adding Webhooks of the github repos to call the VSTS's build REST API.
    - Test report will be generated after the tests are completed.

For developer
    - Phase 1: Manually run/trigger e2e test cases. The e2e can be run locally for the related features or trigger the build definition for the private branch.
    - Phase 2: E2E test can be automatically run when a PR is opened.

### Setup for gated check in
- Source code: The gated check-in source code is located at the folder VSIoT\test\github.
    - app: the source code that integrates the github and VSO
    - test: testing code.
    - config.json properties:
        - builds: build definition for the repo.
        - testspec: Mapping of testing cases and github repos.
- AZure VM:
    - Subscription: VSChina IoT Dev
    - VM: vsts-agent-win
- Deployment
    - The code/app is located on the folder of the VM:  c:\vsciot-app
    - Start redis from the folder c:\vsciot-app\redis in console: redis-server redis.windows.conf
    - Start the app from the folder c:\vsciot-app\VSIoT\test\github: node app.server.js --port=80
    - Need sync the code when the config is changed
- Add webhook:
    - On the setting page of repo, add the the following webhook:
        - Payload URL: http://vsciot-server.westus.cloudapp.azure.com/webhook
        - Secret: VSCIoT
        - Events: Pull request/Status/Push

## Platform independent tests
- Unit test
- Static analysis/Linter
These tests can be run locally and on neutral platform.

## Platform dependent tests
E2E test case: Will be thoroughly test against all the supported systems.

## Test Lib
A wrapper lib for testing hardware will be provided for e2e test cases:
- list: get the boards in the lab that available for run test:
- power on/off: based on the relay system for connection.
- connection: authentication/provisioning

## Hardware Portal(P3)
We should provide a portal to manage the existing hardware:
- List the current status of PC/boards including IP, MAC, status, etc.
- Add remove the hardware to the lab.

## Nightly tests

Nightly test run every nightly, against cleam virtual machines, cross win, linux and osx platforms. they're configured at https://mseng.visualstudio.com/VSIoT/_apps/hub/ms.vss-releaseManagement-web.hub-explorer?definitionId=0&_a=releases.

nightly tests steps:
1. start clean virtual machine
2. install git, nodejs on virtual machine
3. run tests on virtual machine.

nightly test result will send out in email with title "[vsciot-nightly-ubuntu/mac/win]", sample is "[vsciot-nightly-ubuntu][SUCCESS] Run Report #71 [Passrate:100.0 %]"
test run during windows nightly including raspberry pi test and feather m0 wifi tests to broader test coverage.
