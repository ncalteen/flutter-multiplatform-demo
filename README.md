# flutter_multiplatform_demo

An example Flutter project with CI/CD using GitHub Actions and Fastlane.

## Setup

This project uses [Fastlane match](https://docs.fastlane.tools/actions/match/)
to hande certificate management for iOS builds and distribution. Make sure to
read the documentation carefully!

The following sections outline different setup steps needed to use this process
and deploy to Apple/Google app stores. Make sure to read and follow each section
carefully!

### GitHub Actions Configuration

1. Create the following
   [GitHub environments](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-an-environment).
   - `staging`
   - `production`

Throughout the rest of the setup, you will need to add various GitHub Actions
secrets to both environments, as well as repository-level secrets.

> The secret names must match **exactly** as described below. If any secret
> values are the same for both environments, they can instead be created as
> [repository-level secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository).

Make sure to remove any leading/trailing whitespace!

### Amazon Web Services (AWS)

Amazon S3 is used by Fastlane to store signing certificates and provisioning
profiles.

1. Create an Amazon S3 bucket to store Apple Developer certificates and
   provisioning profiles.

   ```bash
   aws s3 mb <bucket-name> \
     --region <region-code>
   ```

1. Ensure developers have read/write access to the bucket in order to manage
   their development keys.
1. Save the following information as GitHub Actions secrets for the repository
   (there's no need to create separate secrets for each repository environment).

   | Secret Name                            | Description                   |
   | -------------------------------------- | ----------------------------- |
   | `FASTLANE_MATCH_AWS_ACCESS_KEY_ID`     | AWS IAM Access Key ID         |
   | `FASTLANE_MATCH_AWS_REGION`            | AWS Region (e.g. `us-east-1`) |
   | `FASTLANE_MATCH_AWS_SECRET_ACCESS_KEY` | AWS IAM Secret Access Key     |
   | `FASTLANE_MATCH_BUCKET_NAME`           | Amazon S3 Bucket Name         |

### Apple Developer Account

1. Create an [Apple Developer Account](https://developer.apple.com/programs/)

   > **Note:** Fastlane recommends a shared account when using `match`. Each
   > developer will have their own local development certificate and
   > provisioning profile, but it allows for easier centralized management.

1. [Register your app](https://developer.apple.com/help/app-store-connect/create-an-app-record/add-a-new-app/)
   on [App Store Connect](https://appstoreconnect.apple.com/)
1. Request access to the
   [App Store Connect API](https://developer.apple.com/app-store-connect/api/)
1. Generate an
   [App Store Connect API key](https://developer.apple.com/documentation/appstoreconnectapi/creating_api_keys_for_app_store_connect_api)
1. Save the following information as GitHub Actions secrets for each
   environment.

   | Secret Name                    | Description                                    |
   | ------------------------------ | ---------------------------------------------- |
   | `APPLE_ID`                     | Apple **App** ID                               |
   | `APPLE_USERNAME`               | **Your** Apple ID / Username                   |
   | `APP_BUNDLE_IDENTIFIER`        | App Bundle Identifier (e.g. `com.example.app`) |
   | `APP_STORE_CONNECT_API_KEY_ID` | App Store Connect API Key ID                   |
   | `APP_STORE_CONNECT_ISSUER_ID`  | App Store Connect Issuer ID                    |
   | `APP_STORE_CONNECT_KEY_FILE`   | App Store Connect Key File **Contents**        |
   | `TEAM_ID`                      | Apple Developer Team ID                        |

   > **Note:** The `APPLE_ID` name is misleading. This should be set to the
   > **App** ID, not a user ID. The Apple App ID can be found in the **App
   > Information** section in **App Store Connect**.

### Fastlane

1. Initialize `match`.

   > Since this project already contains a
   > [Matchfile](./ios/fastlane/Matchfile), you shouldn't need to run this step.
   > Just make sure all the environment variables are set.

   ```bash
   cd ./ios
   fastlane init
   ```

1. Save the following information as GitHub Actions secrets for the repository
   (there's no need to create separate buckets in each environment AWS account).

   | Secret Name               | Description                               |
   | ------------------------- | ----------------------------------------- |
   | `FASTLANE_MATCH_PASSWORD` | Password Set During `fastlane match init` |
   | `FASTLANE_PASSWORD`       | Apple Developer Account Password          |
   | `FASTLANE_USER`           | Apple Developer Account Username          |

### Android Configuration

1. Register on the [Google Play Console](https://play.google.com/console/signup)
1. Generate a signing key, making sure to replace `ALIAS` in the command.

   > **IMPORTANT:** Make sure to save the password, as it will be needed when
   > configuring GitHub Actions.

   ```bash
   sudo keytool -genkey -v \
     -keyalg RSA -keysize 2048 -validity 10000 \
     -keystore ALIAS.keystore \
     -alias ALIAS
   ```

1. Encode the generated `.keystore` file to base64, making sure to replace
   `ALIAS` in the command.

   ```bash
   base64 -i ./ALIAS.keystore -o ./keystore-base64.txt
   ```

1. Create a
   [GCP service account](https://cloud.google.com/iam/docs/service-accounts-create).
1. Generate a
   [service account credentials JSON file](https://cloud.google.com/iam/docs/keys-create-delete).
1. Enable the
   [Google Play Android Developer API](https://console.cloud.google.com/marketplace/product/google/androidpublisher.googleapis.com).
1. Invite the service account to your
   [Google Play Developer account](https://play.google.com/console).
1. Save the following information as GitHub Actions secrets for each
   environment.

   | Secret Name                    | Description                          |
   | ------------------------------ | ------------------------------------ |
   | `ANDROID_KEYSTORE_ALIAS`       | Android Keystore Alias               |
   | `ANDROID_KEYSTORE_FILE_BASE64` | Base64-encoded Android Keystore File |
   |                                | `base64 -i <keystore-name>.keystore` |
   | `ANDROID_KEYSTORE_PASSWORD`    | Android Keystore Password            |
   | `ANDROID_SERVICE_ACCOUNT_JSON` | GCP Service Account Credentials JSON |
   | `ANDROID_SIGNING_KEY_PASSWORD` | Android Signing Key Password         |

> [!IMPORTANT]
>
> Before trying to deploy updates automatically, you must
> [manually upload the first release](https://support.google.com/googleplay/android-developer/answer/9859348?hl=en)
> of the Android app to the Google Play Developer console.

1. After uploading the first app version, save the following information as
   GitHub Actions secrets for each environment.

   | Secret Name            | Description                                   |
   | ---------------------- | --------------------------------------------- |
   | `ANDROID_PACKAGE_NAME` | Android Package Name (e.g. `com.example.app`) |
   | `SUPPLY_PACKAGE_NAME`  | Android Package Name (e.g. `com.example.app`) |

### Local Development

When developing/testing locally, make sure to complete the following steps:

1. [Install Flutter](https://docs.flutter.dev/get-started/install)
1. [Install Fastlane](https://docs.fastlane.tools/)
1. Set the following environment variables.

   | Name | Description |
   | ---- | ----------- |

1. Configure Fastlane match.

   ```bash
   cd ./ios
   fastlane match development
   ```

1. Follow the Flutter/Fastlane integration instructions
   [here](https://docs.flutter.dev/deployment/cd#fastlane).
1. Make sure your app builds locally for Android and iOS.

   ```bash
   # Android
   flutter build appbundle

   # iOS
   flutter build ipa
   ```

   There is a good local development walkthrough in the
   [Flutter documentation](https://docs.flutter.dev/deployment/cd#fastlane).

1. Initialize the Android project

   ```bash
   cd ./android
   fastlane init

   # Follow the on-screen prompts
   ```

1. Initialize the iOS project

   ```bash
   cd ./ios
   fastlane init

   # Follow the on-screen prompts
   ```

## Usage

### `release.yml`

When releasing to testing users, make sure to complete the following:

1. [Distribute the app to beta testers on TestFlight](https://developer.apple.com/documentation/xcode/distributing-your-app-for-beta-testing-and-releases)

## References

- [Installing an Apple certificate on macOS runners for Xcode development](https://docs.github.com/en/actions/deployment/deploying-xcode-applications/installing-an-apple-certificate-on-macos-runners-for-xcode-development)
- [Continuous Delivery with Flutter](https://docs.flutter.dev/deployment/cd#fastlane)

## Build and Publish Workflow

The following workflows are included in this repository:

- [`release.yml`](./.github/workflows/release.yml)

### Workflow Modifications

1. Update the following environment variables.

   | Name                    | Description                                    |
   | ----------------------- | ---------------------------------------------- |
   | `APP_BUNDLE_IDENTIFIER` | App Bundle Identifier (e.g. `com.example.app`) |

### Other

Update this file:
/Users/ncalteen/Workspace/ncalteen/flutter_multiplatform_demo/config/ExportOptions.plist
