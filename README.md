# flutter_multiplatform_demo

An example Flutter project.

## Build and Publish Workflow

The following workflows are included in this repository:

- [`release.yml`](./.github/workflows/release.yml)

## Setup

### Workflow Modifications

1. Update the following environment variables.

   | Name                    | Description                                    |
   | ----------------------- | ---------------------------------------------- |
   | `APP_BUNDLE_IDENTIFIER` | App Bundle Identifier (e.g. `com.example.app`) |

### GitHub Actions Configuration

1. Create the following
   [GitHub environments](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-an-environment).
   - `staging`
   - `production`

Throughout the rest of the setup, you will need to add various GitHub Actions
secrets to both environments.

> The secret names must match **exactly** as described below. If any secret
> values are the same for both environments, they can instead be created as
> [repository-level secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository).

Make sure to remove any leading/trailing whitespace!

### Apple

1. Create an [Apple Developer Account](https://developer.apple.com/programs/)
1. [Register your app](https://developer.apple.com/help/app-store-connect/create-an-app-record/add-a-new-app/)
   on [App Store Connect](https://appstoreconnect.apple.com/)
1. Request access to the
   [App Store Connect API](https://developer.apple.com/app-store-connect/api/)
1. Generate an
   [App Store Connect API key](https://developer.apple.com/documentation/appstoreconnectapi/creating_api_keys_for_app_store_connect_api)
1. Save the following information as GitHub Actions secrets for each
   environment.

   | Secret Name                    | Description                                                           |
   | ------------------------------ | --------------------------------------------------------------------- |
   | `APPLE_ID`                     | Apple ID property in the App Information section in App Store Connect |
   | `APP_STORE_CONNECT_API_KEY_ID` | App Store Connect API Key ID                                          |
   | `APP_STORE_CONNECT_ISSUER_ID`  | App Store Connect Issuer ID                                           |
   | `APP_STORE_CONNECT_KEY_FILE`   | App Store Connect Key File Contents                                   |

1. In the Apple Developer Console,
   [generate an iOS distribution certificate](https://developer.apple.com/help/account/create-certificates/certificates-overview/).

   **Note:** Make sure to select the iOS distribution certificate type!

1. Import the generated `.cer` file into your keychain.
1. Export the certificate as a `.p12` file.
1. Save the following information as GitHub Actions secrets for each
   environment.

   | Secret Name                          | Description                        |
   | ------------------------------------ | ---------------------------------- |
   | `APPLE_SIGNING_CERTIFICATE_BASE64`   | Base64-encoded signing certificate |
   |                                      | `base64 -i <certificate-name>.p12` |
   | `APPLE_SIGNING_CERTIFICATE_PASSWORD` | Signing certificate passphrase     |

1. Generate an
   [App Store Connect provisioning profile](https://developer.apple.com/help/account/manage-provisioning-profiles/create-an-app-store-provisioning-profile).

   **Note:** Make sure to select the iOS Distribution (App Store Connect and Ad
   Hoc) provisioning profile type!

1. Save the following information as GitHub Actions secrets for each
   environment.

   | Secret Name                         | Description                                |
   | ----------------------------------- | ------------------------------------------ |
   | `APPLE_PROVISIONING_PROFILE_BASE64` | Base64-encoded provisioning profile        |
   |                                     | `base64 -i <profile-name>.mobileprovision` |

1. Create a keychain password to use in GitHub Actions. Save this as a GitHub
   Actions secret for each environment.

   | Secret Name               | Description             |
   | ------------------------- | ----------------------- |
   | `APPLE_KEYCHAIN_PASSWORD` | Keychain password value |

### Flutter/Fastlane

When developing/testing locally, make sure to complete the following steps:

1. Install Flutter locally by following
   [their documentation](https://docs.flutter.dev/get-started/install).
1. Install Fastlane locally by following
   [their documentation](https://docs.fastlane.tools/).
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

### Android Configuration

> [!IMPORTANT]
>
> If you haven't done so already, make sure to
> [manually upload the first release](https://support.google.com/googleplay/android-developer/answer/9859348?hl=en)
> of the Android app to the Google Play Developer console. This is required in
> order to perform automated releases later.

1. Register on the [Google Play Console](https://play.google.com/console/signup)
1. Generate a signing key, making sure to replace `YOUR_NAME` in the command.

   > **IMPORTANT:** Make sure to save the password, as it will be needed when
   > configuring GitHub Actions.

   ```bash
   sudo keytool -genkey -v \
     -keyalg RSA -keysize 2048 -validity 10000 \
     -keystore YOUR_NAME.keystore \
     -alias YOUR_NAME
   ```

1. Encode the generated `.keystore` file to base64, making sure to replace
   `YOUR_NAME` in the command.

   ```bash
   base64 -i ./YOUR_NAME.keystore -o ./keystore-base64.txt
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
   | `ANDROID_KEYSTORE_FILE_BASE64` | Base64-encoded Android Keystore File |
   |                                | `base64 -i <keystore-name>.keystore` |
   | `ANDROID_KEYSTORE_PASSWORD`    | Android Keystore Password            |
   | `ANDROID_SERVICE_ACCOUNT_JSON` | GCP Service Account Credentials JSON |
   | `ANDROID_SIGNING_KEY_ALIAS`    | Android Signing Key Alias            |
   | `ANDROID_SIGNING_KEY_PASSWORD` | Android Signing Key Password         |

## Usage

### `release.yml`

When releasing to testing users, make sure to complete the following:

1. [Distribute the app to beta testers on TestFlight](https://developer.apple.com/documentation/xcode/distributing-your-app-for-beta-testing-and-releases)

## References

- [Installing an Apple certificate on macOS runners for Xcode development](https://docs.github.com/en/actions/deployment/deploying-xcode-applications/installing-an-apple-certificate-on-macos-runners-for-xcode-development)
- [Continuous Delivery with Flutter](https://docs.flutter.dev/deployment/cd#fastlane)
