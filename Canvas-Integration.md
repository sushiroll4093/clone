Canvas Integration
======

This page assumes you have followed the [[Production Start]] instructions to get a working Canvas installation.

External Services
-----------

### Amazon S3

You will need an account with [Amazon Web Services](http://aws.amazon.com/) to store data on Amazon S3. Once you have your account, visit your [security credentials](https://aws-portal.amazon.com/gp/aws/developer/account/index.html?action=access-key) page and find your *access key id* and *secret access key*. These two values will help authenticate Canvas to your storage container.

Next, visit the [S3 Management Console](https://console.aws.amazon.com/s3/home) and make sure you have a bucket set up. Note the bucket's name.

To configure Canvas to use your S3 bucket for file storage:

 * Open `config/file_store.yml`, find the *production* section, and change `storage: local` to `storage: s3`, and comment out the `path: tmp/files` line.
 * Open `config/amazon_s3.yml`, find the *production* section, and fill in the *bucket_name*, *access_key_id*, and *secret_access_key* fields in with the values you found out earlier.
 * Restart both Apache and the automated jobs daemon (`sudo /etc/init.d/apache2 restart && sudo /etc/init.d/canvas_init restart`).

Canvas uses Uploadify for file uploading, which in turn uses Flash. If you are uploading files to a different domain than the one being used by your Canvas application, you will need to set up a XML cross-domain policy file in your bucket to avoid sandbox security issues. A typical cross-domain policy file looks like this:

```
<cross-domain-policy>
<site-control permitted-cross-domain-policies="master-only"/>
<allow-access-from domain="yourdomain"/>
</cross-domain-policy>
```

You can read more about cross-domain policy files [here](http://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html).

### Facebook
 * Browse to the Facebook plugin settings at `http://<your-canvas-domain>/plugins/facebook`
 * Follow the instructions there.

### Google Drive

The Google Drive plugin supports the new OAuth 2.0.  To integrate Google Drive, do the following.

 * [Register your instance of Canvas with Google](https://console.developers.google.com).
 * Create a new project or use an existing project
 * Under APIs and auth, modify Redirect URIs as follows:  `https://<your-canvas-domain>/oauth_success?service=google_drive`
 * Make sure Google Drive API is enabled under APIs.
 * Access the google drive plugin and enable:  `https://<your-canvas-domain/plugins/google_drive`
 * Download the JSON from developer console and paste in the box.
 * You may need to also enable Google Docs plugin to enable Google Drive.  You can find your oauth consumer key and secret in the Google Admin Console under Security --> Advanced settings --> Manage Oauth domain key

### Kaltura
You will need an account either at [Kaltura.com](http://www.kaltura.com) or with a self-hosted instance of Kaltura.  

 * Navigate to `http://<your-canvas-domain>/plugins/kaltura`. This is where you'll fill in the values below.
 * Go to the management console and click the **Settings** tab.  
 * Under **Integration Settings** you'll find most of what you need, including *partner id*, *subpartner_id*, *secret_key* (administrator secret) and *user_secret_key*
 * You can specify any *player_ui_conf* that you want.  For custom-built players (check out the **Studio** tab) you can see the id in the UI.
 * For *kcw_ui_conf* and *upload_ui_conf* there's not a good solution to get this in the UI.  If you're hosted on Kaltura.com check out [the Kaltura uploading guide](http://corp.kaltura.com/wiki/index.php/Guides:Upload) and [the uploader widget page](http://www.kaltura.org/kaltura-simple-uploader-ksu-uiconf-and-filetype-filters) for some useful defaults.  If you're self-hosted you probably know how to (or hopefully can figure out how to) set up your own ui_confs.
 * Restart both Apache and the automated jobs daemon (`sudo /etc/init.d/apache2 restart && sudo /etc/init.d/canvas_init restart`).

For more detailed information about setting up Kaltura, please see our [[Kaltura setup instructions]]. Instructure uses a slightly tweaked version of Kaltura v3 available [here](https://s3.amazonaws.com/instructure-kaltura/kalturaCE_v3.0.0-instructure.tar.gz).

### Linked In
 * [Set up a developer accounts with LinkedIn](https://www.linkedin.com/secure/developer) and get your developer API key and secret key.  You'll need to fill out the details of your integration.  You don't need to enter an *OAuth Redirect URL*.
 * Browse to the LinkedIn plugin settings at `http://<your-canvas-domain>/plugins/linked_in`
 * Follow the instructions there.

### Box View

 * [Set up a developer account with Box](https://developers.box.com/view-api/). create a Box Application.  Note the API key at the bottom of the "edit" page for your Box application.
 * Browse to the Canvadocs plugin settings at `http://<your-canvas-domain>/plugins/canvadocs`
 * Enter your API key.
 * The base URL should be set to  https://view-api.box.com/1

### TinyChat

 * Set up an account with TinyChat and [get a developer API key](http://tinychat.com/developer/dashboard/).
 * Browse to the Tinychat plugin settings at `http://<your-canvas-domain>/plugins/tinychat`
 * Follow the instructions there.

### Twitter

 * [Register your instance of Canvas with Twitter](http://dev.twitter.com/apps/new).
 * You'll need to specify the *Registered OAuth Callback URL* as `http://<your-canvas-domain>/oauth_success?service=twitter`.
 * Make a note of your *consumer key* and *consumer secret*.
 * Browse to the Twitter plugin settings at `http://<your-canvas-domain>/plugins/twitter`
 * Follow the instructions there.

### Unsplash

 * [Set up a developer account with Unsplash](https://unsplash.com/developers)
 * Create an Unsplash API application. Please [follow the instructions](https://unsplash.com/documentation#registering-your-application) to have your application approved. If you make note that your application is an instance of Canvas LMS, our team will expedite the approval.
 * Browse to the Unsplash plugin settings at `http://<your-canvas-domain>/plugins/unsplash`
 * Enter your application Access Key and Secret Key