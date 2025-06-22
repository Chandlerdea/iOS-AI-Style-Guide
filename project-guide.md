# Project Guide
This guide specifics my preferred file structure for Xcode projects. By default, Xcode projects do not have this file structure so directories will need to be created and files moved to achieve this preference. Each Xcode project will have a root directory matching the name of the project file. For example, AwesomeApp.xcodeproject will have a root directory named AwesomeApp. This document describes how files should be structured inside of that root directory.

## Important Details
After moving certain files (info.plist, .entitlements, .xcassets, .etc), Xcode will fail to build because the build settings of the project file file needs to be updated with the new paths of these files. Make sure to do this after moving any of these files.

## Defining terms

### Feature
I will use the term "Feature" to describe a screen or multiple screens that encapsulate a piece of functionality inside the app. This will always contain at least one pair of a view and view model. There could be multiple pairs if the feature involves multiple screens. It does not apply to a reused SwiftUI view that does not contain logic (if or switch statements, terinary operators, .etc).

## Project Structure
The following list is the structure of the root directory you should create, where each item is a directory:

- Common
- Extensions
- Models
- Features
- Clients
- Resources
- Supporting Files

Only create these directories, do not create any additional directories.

Below is an explanation of what belongs in each directory:

### Common
This directory contains subdirectories for the Swift types contained in their enclosed Swift files. It should not contain view models for features. It can contain SwiftUI views that do not have a corresponding view model because they are static and resused in multiple features. The directory should only contains subdirectories that classify the type of common components used throughout the app. For example, if there is a `BadgeView` that is used in multiple features, its path would exist in a Views directory.

### Extensions
This directory should contain all files containing only extensions on Swift types. These files will have the naming convention of `TypeBeingExtended+Extensions.swift`.

### Models
This directory should contain all file containing model objects, including any extensions or additions to the model types.

### Features
This directory should contain subdirectories for each feature in the app. If features have SwiftUI views in separate files that are not used outside of one feature, those files should live in the corresponding feature's subdirectory.

### Clients
This directory should contain all clients along with their live implementations. The declaration file ("APIClient.swift) should exist with their live implementations ("APIClient+Live.swift) in the same subdirectory with the name of the provider.

### Resources
This directory should contain all .xcasset files, along with any other resource that is not a source file (.mp4 files, .json file, .etc)

### Supporting Files
This directory should contain any supporting files used by the Xcode project (.xcconfig, info.plist, .entitlement files, .etc).

## SwiftUI apps
For SwiftUI apps, the "ProjectNameApp.swift" file that contains the Swift type conforming to the `App` protocol should exist at the root directory that contains all of the source files for the project.