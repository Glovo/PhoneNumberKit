<div align="center">

# 📚 PhoneNumberKit

**Glovo's fork of the Swift phone-number library · parse · format · validate · as-you-type**

A Swift framework for parsing, formatting and validating international phone numbers, built on
Google's libphonenumber metadata. This repo **is** Glovo's fork of
[marmelroy/PhoneNumberKit](https://github.com/marmelroy/PhoneNumberKit): upstream release 3.3.3
plus five Glovo commits. It **is not** a service and deploys nothing; apps pull it in as a
Swift Package, CocoaPod or Carthage framework.

[![Swift](https://img.shields.io/badge/Swift-5.3_tools-F05138?logo=swift&logoColor=white)](Package.swift)
[![Platforms](https://img.shields.io/badge/platforms-iOS_9_%7C_macOS_10.10_%7C_tvOS_9_%7C_watchOS_2-lightgrey?logo=apple)](Package.swift)
[![SwiftPM](https://img.shields.io/badge/SwiftPM-compatible-F05138?logo=swift&logoColor=white)](Package.swift)
[![CocoaPods](https://img.shields.io/badge/CocoaPods-3.3.6-EE3322?logo=cocoapods&logoColor=white)](PhoneNumberKit.podspec)
[![Carthage](https://img.shields.io/badge/Carthage-compatible-4BC51D)](PhoneNumberKit.xcodeproj/xcshareddata/xcschemes)
<br/>
[![Fork of](https://img.shields.io/badge/fork_of-marmelroy%2FPhoneNumberKit-24292F?logo=github)](https://github.com/marmelroy/PhoneNumberKit)
[![Test files](https://img.shields.io/badge/test_files-4-blue)](PhoneNumberKitTests)
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

</div>

---

## Table of contents

- [Overview](#overview)
- [Adding a libphonenumber metadata update](#adding-a-libphonenumber-metadata-update)
- [Build & local development](#build--local-development)
- [Testing](#testing)
- [Code quality](#code-quality)
- [Repository layout](#repository-layout)
- [Contributing & conventions](#contributing--conventions)
- [Glossary](#glossary)
- [Ownership & support](#ownership--support)

---

## Overview

PhoneNumberKit turns a raw string such as `"+33 6 89 017383"` into an immutable `PhoneNumber`
(country code, national number, extension, type), validates it against libphonenumber's
per-territory rules, and formats it as E.164, international or national. The UIKit part adds
`PhoneNumberTextField`, which formats while the user types.

This fork branched from upstream at release **3.3.3** (`1ef0431`, 2021-01-17). On 2026-10-29
GitHub reports `master` as 5 commits ahead of and 293 commits behind upstream `master`. The
Glovo commits are:

| Change | Ticket | Commit | First tag with it |
|---|---|---|---|
| Updated libphonenumber metadata so new users with a Côte d'Ivoire (+225) number can verify it | `CF-1208` | `c9abd55` | `3.3.4` |
| `PhoneNumber` gets a public memberwise `init` ([`PhoneNumber.swift`](PhoneNumberKit/PhoneNumber.swift)) | — | `b992e27` | `3.3.4` |
| `defaultRegionCode()` reads the Contacts country (`CNContactsUserDefaults`) instead of the deprecated CoreTelephony carrier; on Mac Catalyst `ko` is mapped to `KR` | `RSK-714` | `aaf2495` | `3.3.5` |
| `defaultRegionCode()` returns `ES` when the device region is World (`001`) | `RSK-730` | `a675b92` | none (podspec says `3.3.6`, no tag) |

| | |
|---|---|
| **Language / build** | Swift · SwiftPM tools 5.3 ([`Package.swift`](Package.swift)) · `swift_version` 5.0 ([`PhoneNumberKit.podspec`](PhoneNumberKit.podspec)) |
| **Platforms** | iOS 9 · macOS 10.10 · tvOS 9 · watchOS 2 |
| **Distribution** | SwiftPM · CocoaPods (podspec `3.3.6`) · Carthage (shared Xcode schemes) |
| **Data** | Bundled [`PhoneNumberMetadata.json`](PhoneNumberKit/Resources/PhoneNumberMetadata.json), generated from libphonenumber's XML |
| **Default region** | Contacts country → `Locale` country → `US` ([`PhoneNumberKit.swift`](PhoneNumberKit/PhoneNumberKit.swift), [`Constants.swift`](PhoneNumberKit/Constants.swift)) |

| Product | Contents |
|---|---|
| SwiftPM `PhoneNumberKit` (also `PhoneNumberKit-Static`, `PhoneNumberKit-Dynamic`) | The whole [`PhoneNumberKit/`](PhoneNumberKit) target, UI included, plus the metadata JSON |
| Pod subspec `PhoneNumberKit/PhoneNumberKitCore` | `PhoneNumberKit/*.swift` plus the metadata JSON |
| Pod subspec `PhoneNumberKit/UIKit` | [`PhoneNumberKit/UI/`](PhoneNumberKit/UI): `PhoneNumberTextField`, `CountryCodePickerViewController` |

<details><summary><b>Features (from the upstream README)</b></summary>

| | Feature |
|---|---|
| :phone: | Validate, normalize and extract the elements of any phone number string. |
| :100: | Simple Swift syntax and a lightweight readable codebase. |
| :checkered_flag: | Fast. 1000 parses -> ~0.4 seconds. |
| :books: | Best-in-class metadata from Google's libPhoneNumber project. |
| :trophy: | Fully tested to match the accuracy of Google's JavaScript implementation of libPhoneNumber. |
| :iphone: | Built for iOS. Picks the default region from the device (see the Default region row above). |
| 📝 | Editable (!) AsYouType formatter for UITextField. |
| :us: | Convert country codes to country names and vice versa. |

</details>

### Install

> [!WARNING]
> [`PhoneNumberKit.podspec`](PhoneNumberKit.podspec) (`s.source`, `s.homepage`) and the upstream
> install snippets point at **marmelroy/PhoneNumberKit**. Installing from there gives you upstream
> code **without** the Glovo changes above. Point at `https://github.com/Glovo/PhoneNumberKit.git`.

Tags in this fork go up to `3.3.5`. The `RSK-730` change (`a675b92`) is only on `master`.

```swift
// Package.swift
dependencies: [
    .package(url: "https://github.com/Glovo/PhoneNumberKit.git", .exact("3.3.5"))
    // or, to include RSK-730: .package(url: "https://github.com/Glovo/PhoneNumberKit.git", .revision("a675b92591f0a156bbdf9e1b29771003f812ad56"))
]
```

```ruby
# Podfile
pod 'PhoneNumberKit', :git => 'https://github.com/Glovo/PhoneNumberKit.git', :tag => '3.3.5'
```

```ogdl
# Cartfile
github "Glovo/PhoneNumberKit" == 3.3.5
```

<details><summary><b>Upstream install instructions (install marmelroy's release, not this fork)</b></summary>

**[Preferred] Setting up with [Swift Package Manager](https://swiftpm.co/?query=PhoneNumberKit).**
The [Swift Package Manager](https://swift.org/package-manager/) is the preferred tool for distributing PhoneNumberKit. From Xcode 11+:

1. Select File > Swift Packages > Add Package Dependency. Enter `https://github.com/marmelroy/PhoneNumberKit.git` in the "Choose Package Repository" dialog.
2. In the next page, specify the version resolving rule as "Up to Next Major" with "3.3.3".
3. After Xcode checked out the source and resolving the version, you can choose the "PhoneNumberKit" library and add it to your app target.

For more info, read [Adding Package Dependencies to Your App](https://developer.apple.com/documentation/xcode/adding_package_dependencies_to_your_app) from Apple.
Alternatively, add PhoneNumberKit to your `Package.swift` file:

```swift
dependencies: [
    .package(url: "https://github.com/marmelroy/PhoneNumberKit", .upToNextMajor(from: "3.3.3"))
]
```

**Setting up with Carthage.** [Carthage](https://github.com/Carthage/Carthage) is a decentralized dependency manager that automates the process of adding frameworks to your Cocoa application. Install it with [Homebrew](http://brew.sh/):

```bash
brew update
brew install carthage
```

Then specify it in your `Cartfile`:

```ogdl
github "marmelroy/PhoneNumberKit"
```

**Setting up with [CocoaPods](http://cocoapods.org/?q=PhoneNumberKit).**

```ruby
source 'https://github.com/CocoaPods/Specs.git'
pod 'PhoneNumberKit', '~> 3.3'
```

</details>

### Usage

Import PhoneNumberKit at the top of the Swift file that works with phone numbers, then allocate
a `PhoneNumberKit` object. All interaction goes through it.

> [!IMPORTANT]
> A `PhoneNumberKit` instance is **expensive to allocate**: it parses the metadata and keeps it in
> memory for the object's lifecycle. Allocate it **once** and deallocate it when no longer needed.

```swift
import PhoneNumberKit

let phoneNumberKit = PhoneNumberKit()
```

To parse a string, use `parse`. The region code is computed automatically (see the Default
region row above) but can be overridden. Parsing does a hard type validation by default; this
is costly and can be turned off with `ignoreType: true`.

```swift
do {
    let phoneNumber = try phoneNumberKit.parse("+33 6 89 017383")
    let phoneNumberCustomDefaultRegion = try phoneNumberKit.parse("+44 20 7031 3000", withRegion: "GB", ignoreType: true)
}
catch {
    print("Generic parser error")
}
```

To parse and validate many numbers at once, use the array function. Invalid numbers are left
out of the result.

```swift
let rawNumberArray = ["0291 12345678", "+49 291 12345678", "04134 1234", "09123 12345"]
let phoneNumbers = phoneNumberKit.parse(rawNumberArray)
let phoneNumbersCustomDefaultRegion = phoneNumberKit.parse(rawNumberArray, withRegion: "DE",  ignoreType: true)
```

`PhoneNumber` objects are immutable Swift structs:

```swift
phoneNumber.numberString
phoneNumber.countryCode
phoneNumber.nationalNumber
phoneNumber.numberExtension
phoneNumber.type // e.g Mobile or Fixed
```

Formatting a `PhoneNumber` into a string:

```swift
phoneNumberKit.format(phoneNumber, toType: .e164) // +61236618300
phoneNumberKit.format(phoneNumber, toType: .international) // +61 2 3661 8300
phoneNumberKit.format(phoneNumber, toType: .national) // (02) 3661 8300
```

Query the countries for a dialing code, or the dialing code for a country:

```swift
phoneNumberKit.countries(withCode: 33)
phoneNumberKit.countryCode(for: "FR")
```

<details><summary><b>PhoneNumberTextField (as-you-type formatting)</b></summary>

![AsYouTypeFormatter](https://user-images.githubusercontent.com/7651280/67554038-e6512500-f751-11e9-93c9-9111e899a2ef.gif)

To use the AsYouTypeFormatter, replace your `UITextField` with a `PhoneNumberTextField`. If you
use Interface Builder, make sure the module field is set to PhoneNumberKit.

You can customize the text field UI:

- `withFlag` displays the country code for the `currentRegion`. The `flagButton` is shown in the `leftView` of the text field, sized from your text size.
- `withExamplePlaceholder` uses `attributedPlaceholder` to show an example number for the `currentRegion`. When `withPrefix` is also set, the country code prefix is inserted and removed automatically while editing.

`PhoneNumberTextField` formats numbers automatically and gives the user full editing
capabilities. To customize further, use `PartialFormatter` directly. The default region is
computed automatically but can be overridden by subclassing (the `defaultRegion` setter is
deprecated):

```swift
class MyGBTextField: PhoneNumberTextField {
    override var defaultRegion: String {
        get {
            return "GB"
        }
        set {} // exists for backward compatibility
    }
}
```

```swift
let textField = PhoneNumberTextField()

PartialFormatter().formatPartial("+336895555") // +33 6 89 55 55
```

</details>

<details><summary><b>Need more customization? Access the metadata directly</b></summary>

You can access the metadata powering PhoneNumberKit yourself and program any behaviour the way
PhoneNumberKit does internally. You are then exposed to the less polished interface of the
underlying file format.

```swift
phoneNumberKit.metadata(for: "AU")?.mobile?.exampleNumber // 412345678
```

</details>

---

## Adding a libphonenumber metadata update

The metadata lives in [`PhoneNumberMetadata.json`](PhoneNumberKit/Resources/PhoneNumberMetadata.json).
It is generated from the original XML at
[`Original/PhoneNumberMetadata.xml`](PhoneNumberKit/Resources/Original/PhoneNumberMetadata.xml)
by [`update.sh`](PhoneNumberKit/Resources/update.sh). Full notes:
[`PhoneNumberKit/Resources/README.md`](PhoneNumberKit/Resources/README.md).

> [!WARNING]
> Glovo's `CF-1208` commit (`c9abd55`) edited both the XML and the JSON by hand. Replacing the
> XML with a fresh libphonenumber copy **replaces that edit too**. Check that Côte d'Ivoire
> (+225) numbers still parse before you ship.

1. Download a newer XML metadata file from [libPhoneNumber](https://github.com/googlei18n/libphonenumber/blob/master/resources/).
2. Replace [`PhoneNumberKit/Resources/Original/PhoneNumberMetadata.xml`](PhoneNumberKit/Resources/Original/PhoneNumberMetadata.xml) with it.
3. Install the converter and regenerate the JSON. `update.sh` reads `$(pwd)/Original/…`, so run it from `PhoneNumberKit/Resources`:

   ```bash
   pip install xmljson                 # python library used by update.sh
   cd PhoneNumberKit/Resources
   ./update.sh                         # XML → PhoneNumberMetadata.json (minified, newlines stripped)
   ```

4. Run the tests (see [Testing](#testing)).
5. Bump `s.version` in [`PhoneNumberKit.podspec`](PhoneNumberKit.podspec); every Glovo release commit so far did.

---

## Build & local development

> [!NOTE]
> **Prerequisites:** a full **Xcode** install. `swift build` works with Command Line Tools only
> (checked with Swift 6.4 on 2026-10-29), but the tests need XCTest, which ships with Xcode.
> Python with `xmljson` is only needed for metadata updates. CocoaPods is only needed to lint the
> podspec.

```bash
swift build                          # SwiftPM build of the PhoneNumberKit target
./build.sh build                     # xcodebuild build + test, scheme PhoneNumberKit, iOS Simulator
./build.sh examples                  # build every sample app under examples/
pod lib lint PhoneNumberKit.podspec  # validate the podspec (from the podspec's header comment)
open PhoneNumberKit.xcodeproj        # schemes: PhoneNumberKit (iOS), -macOS, -tvOS, -watchOS
```

### Run locally

There is no app to run. The two sample apps,
[`examples/AsYouType`](examples/AsYouType) and [`examples/PhoneBook`](examples/PhoneBook), each
have a `Sample.xcodeproj`.

> [!NOTE]
> Both sample projects resolve PhoneNumberKit from **upstream** `https://github.com/marmelroy/PhoneNumberKit.git`
> (up to next major from 3.3.0, pinned at 3.3.0), so they do **not** exercise this fork's code.

### Build logic

- **SwiftPM** ([`Package.swift`](Package.swift)) excludes `Resources/Original`, `Resources/README.md`, `Resources/update.sh`, `Info.plist` and `Bundle+Resources.swift` from the target and processes `PhoneNumberMetadata.json` as a resource.
- [`Bundle+Resources.swift`](PhoneNumberKit/Bundle+Resources.swift) provides `Bundle.module` for the non-SwiftPM builds (CocoaPods, Carthage, Xcode project); SwiftPM generates its own.
- **CocoaPods** ([`PhoneNumberKit.podspec`](PhoneNumberKit.podspec)) has two subspecs, `PhoneNumberKitCore` and `UIKit`, with `swift_version` 5.0.
- The podspec (`s.ios.frameworks`, `s.osx.frameworks`) and the Xcode project still link **CoreTelephony**, although no source file imports it since `RSK-714`.

---

## Testing

```bash
./build.sh build      # build + test on "platform=iOS Simulator,OS=14.0,name=iPhone 11 Pro Max"
swift test            # SwiftPM test target PhoneNumberKitTests (needs Xcode for XCTest)
```

- **4 test files** in [`PhoneNumberKitTests/`](PhoneNumberKitTests): `PartialFormatterTests`, `PhoneNumberKitParsingTests`, `PhoneNumberKitTests` and `PhoneNumberTextFieldTests` (the last one only compiles where UIKit is available).
- The simulator destination is hard-coded in [`build.sh`](build.sh) with the comment "Update me when new Xcode versions are released!". Change `PLATFORM` if your Xcode doesn't have that simulator.
- None of the four Glovo changes in [Overview](#overview) has a test.
- The only CI config is [`.travis.yml`](.travis.yml), inherited from upstream: on pushes to `master` it runs `./build.sh build` and `./build.sh examples` on `osx_image: xcode12`. There are no GitHub Actions workflows, and commit `a675b92` on `master` has no status or check run.

---

## Code quality

| Tool | Role |
|---|---|
| SwiftLint ([`.swiftlint.yml`](.swiftlint.yml)) | Lint rules. Many rules are disabled, marked "should be removed from disabled_rules, code needs a lot of refactoring" |
| SwiftLint ([`.swiftlint.auto.yml`](.swiftlint.auto.yml)) | A second, more permissive rule set; no script references it |
| SwiftFormat ([`.swiftformat`](.swiftformat)) | Formatting: 4-space indent, alphabetized imports, explicit `self` (`--self insert`), LF line breaks |

Neither tool runs from `build.sh`, Travis, the Xcode project or a git hook. They only run when
you run them from the repo root:

```bash
swiftlint             # picks up .swiftlint.yml
swiftformat .         # picks up .swiftformat
```

---

## Repository layout

```text
.
├── PhoneNumberKit/                       ⭐ library source (SwiftPM target "PhoneNumberKit")
│   ├── PhoneNumberKit.swift              public entry point: parse, format, countries, defaultRegionCode()
│   ├── PhoneNumber.swift                 immutable PhoneNumber struct
│   ├── ParseManager.swift · PhoneNumberParser.swift · Formatter.swift · PartialFormatter.swift
│   ├── MetadataManager.swift · MetadataParsing.swift · MetadataTypes.swift · RegexManager.swift
│   ├── UI/                               PhoneNumberTextField, CountryCodePickerViewController (pod subspec UIKit)
│   └── Resources/
│       ├── PhoneNumberMetadata.json      bundled metadata, generated by update.sh
│       ├── Original/                     libphonenumber source XML (edited by CF-1208)
│       ├── update.sh                     XML → JSON converter
│       └── README.md                     metadata update notes
├── PhoneNumberKitTests/                  XCTest suite (4 test files)
├── examples/                             AsYouType and PhoneBook sample apps (use upstream PhoneNumberKit)
├── Documentation/OXMIGRATIONGUIDE.md     0.x → 1.0 migration guide
├── PhoneNumberKit.xcodeproj/             Xcode project, shared schemes for iOS, macOS, tvOS, watchOS
├── Package.swift                         SwiftPM manifest
├── PhoneNumberKit.podspec                CocoaPods spec (version 3.3.6, source points at upstream)
├── build.sh                              build + test, or build the examples
├── .travis.yml                           Travis config inherited from upstream
├── .swiftlint.yml · .swiftlint.auto.yml · .swiftformat
└── LICENSE                               MIT
```

---

## Contributing & conventions

1. **Push generic fixes upstream.** The previous README asked for useful changes to go upstream; all five Glovo commits so far landed on `master` of this fork.
2. **Ticket keys in commit subjects:** Glovo commits reference Jira keys, e.g. `[CF-1208] …`, `RSK-714 …`, `[RSK-730] …`.
3. **New non-Swift files under `PhoneNumberKit/`** need an `exclude:` or `resources:` entry in [`Package.swift`](Package.swift), like the existing ones.
4. **Breaking-API history:** the 0.x → 1.0 changes are in [`Documentation/OXMIGRATIONGUIDE.md`](Documentation/OXMIGRATIONGUIDE.md).

There is no `CODEOWNERS` file, so GitHub assigns no code-owner reviewers.

---

## Glossary

| Term | Meaning |
|---|---|
| **E.164** | International format with no spaces, e.g. `+61236618300` (`.e164`) |
| **Region code** | Two-letter CLDR region ID such as `GB` or `FR`; `001` is the non-geographical "World" entity |
| **Country code** | Numeric calling code, e.g. `33` for `FR` (`countryCode(for:)`) |
| **National number** | The number without its country code (`PhoneNumber.nationalNumber`) |
| **libphonenumber metadata** | Google's per-territory patterns and formats, bundled here as JSON |
| **As-you-type formatting** | Formatting while the user types, via `PartialFormatter` or `PhoneNumberTextField` |

---

## Ownership & support

- **Team:** none declared. The repo has no `CODEOWNERS` file and no `catalog-info.yaml`.
- **Last Glovo change:** `a675b92` (`RSK-730`) on 2023-07-24.
- **Upstream:** generic library bugs belong to [marmelroy/PhoneNumberKit](https://github.com/marmelroy/PhoneNumberKit).
- **License:** MIT, see [`LICENSE`](LICENSE).
