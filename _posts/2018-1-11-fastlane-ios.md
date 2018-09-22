---
layout: post
tags: ios fastlane
title: Easy CI with fastlane - How we automated everything iOS from running tests to distributing a build
---
## Easy CI with fastlane: How we automated everything iOS from running tests to distributing a build

### Where We Were
Developers ran unit tests locally.
Reviewers on PR couldn’t see results of unit tests, or other code quality information.

Engineers adding a new testing device needed to click around the Apple Developer Portal. A lot.

Testers didn’t know when new builds were ready — and sometimes needed to use external tools like Diawi to install build.

Internal users couldn’t use dogfooding builds because of issues when manually updating provisioning profiles.

We decided that our first priority was to focus on implementing continuous integration for our iOS and Apple Watch apps (as well as lay the groundwork for eventually implementing CI for our Android apps).

### fastlane to the rescue!
We chose fastlane as our CI tool of choice for the following reasons:

- It’s an open-source toolset with tremendous and outstanding support.
- It’s incredibly flexible and lets us automate iOS specific tasks like certificate management, automatically incrementing build numbers, and device provisioning.
- It’s portable so we won’t get vendor-locked. fastlane isn’t coupled to a particular CI platform, so we can easily migrate our fastlane implementation to a different SaaS provider or our own self-hosted CI.

### Before you get started
You’ll want to sort out a small (but important!) list of things you’ll need. This will make your fastlane implementation as frictionless as possible — but also ensures your CI process will be maintainable for a long time to come.

You’ll need:

- An Email Account — We’ll be creating a bunch of accounts (and even more if you’re tackling Android CI too!). A good email (like ios-ci@yourcompany.com) helps to keep things organized.
- A Github (or [INSERT SCM HERE]) Account with SSH Key Pair — You’ll need this for your CI system. If you want to automatically increment the build number, the SSH key pair will also automatically commit changes.
- An Apple Developer Admin account — fastlane Match handles all the messy provisioning business for you — you just need to give it the account!
- Password Management — When you’re done, you’ll have something like three accounts, a SSH key pair, and a match repository password. To set up fastlane for local development, you’ll need access to the Apple Developer account and the match repository password. We use [Lastpass Enterprise](https://www.lastpass.com/enterprise) to securely share passwords with the team, though you can also use free alternatives or even secure messaging platforms.

### SaaS versus Device Lab

astlane is working on a mobile-focused, open source CI tool. This is exciting news and we’ll be excited to take fastlane.ci out for a spin in late 2018.

In the meantime, you’ll need to choose between running your mobile CI in-house (using Jenkins or Atlassian Bamboo) or with a cloud-based SaaS provider (like CircleCI, TravisCI, or Gitlab).

*Self-Hosted*
##### Pros

- Super flexible! You have control over how your mobile CI lab plugs into the rest of your company’s infrastructure — which can be particularly important if you work in a highly-regulated industry.
- You can centralize all (ALL) your builds in a single location. Mobile doesn’t have to be a special snowflake!
##### Cons

- Number of parallel builds is limited by number of physical devices.
- Some solutions (i.e. Jenkins) are dependent on infrequently maintained plugins.
- High initial cost and ongoing opportunity cost for ongoing maintenance.

*SaaS*
##### Pros

- There’s a smaller infrastructure “lift” to start running mobile CI — making adoption easier and more frictionless.
- Build environment is more stable (both in terms of infrastructure and updates).
##### Cons

- Troubleshooting Xcode/code signing issues is difficult.
- Depending on the platform and team practices, monthly fees can become expensive over time.
### What we’re doing
![alt text](https://github.com/naviat/naviat.github.io/blob/master/images/fastlane.png "Flow")
1. When a developer opens a PR, CircleCI will automatically build the iOS project.
1. CircleCI will automatically install code signing certificates and provisioning profiles into a local keychain (configured using the setup_circle_ci action). For more information on fastlane and code signing, see the match action documentation and read more about the approach here.
1. fastlane runs SwiftLint, runs unit tests, and generates code coverage reports.
1. fastlane builds an IPA and *.dsym files.

We can choose to distribute the build via AWS using the [fastlane-plugin-s3](https://github.com/joshdholtz/fastlane-plugin-s3) addon (for QA) or Crashlytics (for internal users).

We’re using fastlane’s slack support to notify our team when things go off the rails (e.g. build fails) or when they succeed beyond our wildest dreams (i.e. a build is ready for QA to test). There’s definitely a risk of alarm fatigue here — work with your team to find out how many notifications are “just right”!

### What we learned
#### Xcode schemes for Develop, Beta, and Production: One Xcode project config for all
To facilitate our day-to-day development activities as well as our nightly beta builds, we needed to modify our Xcode project scheme structure to play nice with fastlane match.

We organized our Xcode project with three separate build configurations:

- Debug: For local development (points to our staging environment). Provisioned with a Development profile and Development cert in order for engineers to build to development devices.
- Beta: For internal distribution (also points to our staging environment). Provisioned with an Ad Hoc profile and Distribution cert in order to build and deploy for internal dogfooding.
- Release: For archiving a production build to submit to the App Store.
First, we deactivated `Automatically manage signing` in Xcode.We then set the provisioning profiles in the Xcode Signing (located under the General tab) to each respective match provisioning profile for Development and Ad Hoc. By using match along with a shared Xcode provisioning setup, we ensure that iOS provisioning and code signing is handled consistently (and reproducibly!)
We then created three separate Xcode schemes, one for each build configuration, and we ensure that these schemes are checked into version control by checking the `Shared` box (in the Scheme edit window) when creating a new scheme in Xcode. The Scheme’s run configuration is then set to its respective build configuration (i.e. Debug, Beta, or Release).
Now with this shared Xcode scheme structure, each engineer will only have to run fastlane match to ensure local provisioning profiles are up to date:

```shell
$ fastlane match development --readonly
```
And for our mobile CI, we can easily specify fastlane to deploy an Ad Hoc build with the Beta scheme without ever having to specify any additional Xcode configuration.

#### Setting up fastlane match for WatchKit extension
n addition to our main iOS application target, our project contains a Watch and WatchExtension targets for our Apple Watch app. We use fastlane match to generate both a Development and an Ad Hoc provisioning profile for each of the Watch targets, and fastlane match is effortlessly able to handle provisioning profile generation for [multiple Xcode targets](https://docs.fastlane.tools/actions/match/#handle-multiple-targets).
n the same way we set up Xcode signing for our main application target, we deactivate `Automatically manage signing` for both the Watch and WatchExtension targets, and we set the Provisioning Profile to each respective match profile for Debug, Beta, and Release.

We can now build and deploy our iOS and Apple Watch apps using fastlane gym. If we want to deploy an Ad Hoc build with the Beta run configuration, we can simply configure a `beta` lane to run the gym command:
```
lane :beta do
  gym(scheme: "Beta", 
      configuration: "Beta", 
      export_method: "ad-hoc"
  )
end
```
Tip: When using the useful (and recommended!) increment_build_number action, ensure that the Xcode project plist paths for the Watch and Watch Extension targets are relative paths in order to facilitate plist editing by making sure `$(SRCROOT)` is removed from the plist paths specified in the Build Settings for both Watch targets.

#### Unit Tests, Code Coverage, and Notification strategy
Testing is an important and valuable step in the development cycle across our products and processes here at Aaptiv, including both our iOS and Android codebases. Specifically, for our iOS unit test suite, we add logic tests to defend against introducing regressions with a new branch as well as providing additional validation when we refactor components within the codebase.

We use fastlane scan to run our XCTest suite against an open PR, which will then report both the results of the unit tests and code coverage. In order to run unit tests and generate reports, we simply wrap a call to the scan command in a test lane in our Fastfile. The actual scan command parameters are minimal since we set our parameter defaults in the Scanfile including the Xcode scheme, an array of device type strings, and output types for the generated reports.

With this feedback loop in place, when and how exactly should we surface unit test results while avoiding alarm fatigue?
For starters, we created a Slack channel specifically for iOS-related test and build notifications, and a corresponding Slack webhook URL that we can plug into fastlane’s built-in slack action using the fastlane SLACK_URL environment variable. Next, we as a team decided upon a notification strategy that results in a notification being sent to our iOS notifications channel when:

- Unit test failure
- Nightly build failure/success
- An error occurred in a lane
This notification strategy has worked well for our team by only propagating alerts for the scenarios we are interested in and surfacing the notifications in a centralized channel. Of course this notification strategy will vary from team to team depending upon the team’s workflow.

Additionally, we ensure that notifications are only broadcasted when fastlane is run within our mobile CI using fastlane’s is_ci action (we wouldn’t want to send a notification each time we run unit tests locally!)

In order to limit when notifications are posted for unit test results, the scan tool has two very convenient parameters:

- `skip_slack` which we use to prevent posting notifications when unit tests are run locally.
- `slack_only_on_failure` which we use to only post a notification if unit tests fail while being run in our mobile CI.

With a thought out notification strategy in place, we now can funnel success and failure results to Slack as well as surfacing the unit test status within a opened PR. 
Tip: Is `SwiftLint erroring` on the `fastlane.swift` file? Exclude the Bundle path i.e. vendor/bundle in your `swiftlint.yml` config file by adding:
```
excluded:
  - vendor/bundle
```

You will also want to add the CocoaPods path as well to exclude SwiftLint from erroring on any dependencies that are built with your project:

```
excluded:
  - Pods
```
### Best Practices

There is a wealth of resources in both the (outstanding!) official fastlane documentation as well as various points of knowledge on the web. For our development here at Aaptiv, the following best practices helped us in developing our fastlane and CI solution:

- Single Responsibility for Lanes: Ensure that each Fastfile lane has a single responsibility. As additional requirements are added or you want to add a step into your pipeline, treat these lanes as individual units which can then be linked together in your CI workflow configuration. This will provide flexibility in the future without bloating individual lanes (and spending less time refactoring or reworking the lanes in the future!)
- Specify defaults outside of the Fastfile: Reduce clutter in the Fastfile, by specifying param defaults for deliver, gym, match, and scan in the respective Deliverfile, Gymfile, Matchfile, and Scanfile.
- Gemfile in your repo: We recommend adding a Gemfile to the root of your iOS project to ensure all local development environments specify and use the same fastlane gem version.
- Fastfile Discoverability: Add descriptions to each lane for extra discoverability. This can be as succinct as just the single responsibility for that one lane (i.e. “Builds ad-hoc IPA file” or “Increments iOS build number”):
```
desc("Increments iOS build number")
lane :build_bump do
     ...
end
```
### Where We Go From Here
Over our last major project (running about two and a half months), fastlane helped us code, test, and release with confidence. In total, we used our CI process to build ten ad-hoc releases, twenty-two nightly releases, and four beta releases.

While we’re very happy with our CI process, there’s always room to improve! We have some enhancements we’d like to tackle over the next few months:

- Automating app submission to the Apple App Store.
- Allowing developers to add release notes for internal builds so our beta testers will always know what’s going on!
- Speeding up project build times! (caching CocoaPods, investigating the new Xcode Build System, caching project files and only building changed files)
