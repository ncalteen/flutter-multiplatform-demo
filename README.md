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

1. Create an [Apple Developer Account](https://developer.apple.com/programs/).

   > **Note:** Fastlane recommends a shared account when using `match`. Each
   > developer will have their own local development certificate and
   > provisioning profile, but it allows for easier centralized management.

1. [Register your app](https://developer.apple.com/help/app-store-connect/create-an-app-record/add-a-new-app/)
   on [App Store Connect](https://appstoreconnect.apple.com/).
1. Request access to the
   [App Store Connect API](https://developer.apple.com/app-store-connect/api/).
1. Generate an
   [App Store Connect API key](https://developer.apple.com/documentation/appstoreconnectapi/creating_api_keys_for_app_store_connect_api).
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

1. Register on the
   [Google Play Console](https://play.google.com/console/signup).
1. Generate your upload key and keystore using
   [Android Studio](https://developer.android.com/studio/publish/app-signing#generate-key).

   > **Note:** Make sure to save the keystore path, keystore password, upload
   > key alias, and upload key password.

1. Encode the generated keystore file to base64.

   ```bash
   base64 -i KEYSTORE_PATH -o ./upload-keystore-base64.txt
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
   | `ANDROID_KEYSTORE_FILE_BASE64` | Base64-Encoded Android Keystore File |
   | `ANDROID_KEYSTORE_PASSWORD`    | Android Keystore Password            |
   | `ANDROID_SERVICE_ACCOUNT_JSON` | GCP Service Account Credentials JSON |
   | `ANDROID_SIGNING_KEY_ALIAS`    | Android Signing Key Alias            |
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

If the keystore gets lost or needs to be replaced, follow the below steps:

1. Generate a new keystore.
1. Export your certificate.

   ```bash
   keytool -export -rfc -keystore KEYSTORE.jks -alias ALIAS -file CERTIFICATE.pem
   ```

1. Request an upload key reset, providing the exported key.

   > **Note:** This can take 1-2 days to be completed!

### Files to Update

The following files and projects will need to be updated when using this for
your own project.

- [`ios/config/ExportOptions.plist`](./ios/config/ExportOptions.plist)
  - Update the provisioning profile to the one used by Fastlane match.
- TODO: Finish list

### Workflow Modifications

In the [`release.yml`](./.github/workflows/release.yml), update the following:

| Name                    | Description                 |
| ----------------------- | --------------------------- |
| `ANDROID_PACKAGE_NAME`  | Android Package Name        |
| `APP_BUNDLE_IDENTIFIER` | Apple App Bundle Identifier |
| `SUPPLY_PACKAGE_NAME`   | Android Package Name        |

## Local Development

When developing/testing locally, make sure to complete the following steps:

1. [Install Flutter](https://docs.flutter.dev/get-started/install)
1. [Install Fastlane](https://docs.fastlane.tools/)
1. Copy [`.env.example`](./.env.example) to `.env` and fill out the values
1. Load the `.env` into your environment variables

   ```bash
   source .env
   ```

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

## Build and Publish Workflow ([`release.yml`](./.github/workflows/release.yml))

The relase workflow handles the bulk of building and publishing for Android and
iOS. It supports the following functionality:

- **Triggers:** Manual (`workflow_dispatch`) and creation of a release.
- **Functionality:**
  - Runs unit tests
  - Builds and signs app bundles
  - Deploys to Android and Google Play
- **Notes:**
  - The workflow uses GitHub Actions environments to track different settings
    and secrets needed to deploy the app bundles
  - When triggered by a release event, it defaults to **production** releases
    (e.g. a new version of the app is deployed to the public app stores).
  - When triggered manually, it defaults to **staging** relases (e.g. release to
    TestFlight and Google Play internal track).

## References

- [Installing an Apple certificate on macOS runners for Xcode development](https://docs.github.com/en/actions/deployment/deploying-xcode-applications/installing-an-apple-certificate-on-macos-runners-for-xcode-development)
- [Continuous Delivery with Flutter](https://docs.flutter.dev/deployment/cd#fastlane)
