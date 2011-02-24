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

### Facebook
 * [Create a new Facebook App](http://www.facebook.com/developers/).
 * In the **Web Site** section make a note of your *Application ID* and *Application Secret*
 * In the **Facebook Integration** section set *Canvas URL* to `http://<your-canvas-domain>/facebook`
 * Set *Post-Authorize Callback URL* to `http://<your-canvas-domain>/facebook/add_user`
 * Set *Post-Authorize Redirect URL* to `http://<your-canvas-domain>/facebook/authorize_user`
 * Make a note of your *Canvas Page* value
 * Set *Canvas Type* to `FBML`
 * In the **Advanced** section set *Deauthorize Callback* to `http://<your-canvas-domain>/facebook/remove_user`
 * Copy `config/facebooker.yml.example` to `config/facebooker.yml` and open it up. Find the *production* section of the configuration file.
 * Replace `api_key: key` with `api_key: <your_application_id>`
 * Replace `secret_key: secret_key` with `secret_key: <your_application_secret>`
 * Replace `canvas_page_name: page_name` with `canvas_page_name: <your_canvas_page>`
 * Restart both Apache and the automated jobs daemon (`sudo /etc/init.d/apache2 restart && sudo /etc/init.d/canvas_init restart`).

### Google Docs

 * [Register your instance of Canvas with Google](https://www.google.com/accounts/ManageDomains).
 * Copy `config/google_docs.yml.example` to `config/google_docs.yml` and open it up. Find the *production* section of the configuration file and input the OAuth consumer key and OAuth consumer secret registering with Google gave you.
 * Restart both Apache and the automated jobs daemon (`sudo /etc/init.d/apache2 restart && sudo /etc/init.d/canvas_init restart`).

### Kaltura
You will need an account either at [Kaltura.com](http://www.kaltura.com) or with a self-hosted instance of Kaltura.  

 * Copy `config/kaltura.yml.example` to `config/kaltura.yml` and open it up.  Find the *production* section of the configuration file.  This is where you'll fill in the values below.
 * Go to the management console and click the **Settings** tab.  
 * Under **Integration Settings** you'll find most of what you need, including *partner id*, *subpartner_id*, *secret_key* (administrator secret) and *user_secret_key*
 * You can specify any *player_ui_conf* that you want.  For custom-built players (check out the **Studio** tab) you can see the id in the UI.
 * For *kcw_ui_conf* and *upload_ui_conf* there's not a good solution to get this in the UI.  If you're hosted on Kaltura.com check out [the Kaltura uploading guide](http://corp.kaltura.com/wiki/index.php/Guides:Upload) and [the uploader widget page](http://www.kaltura.org/kaltura-simple-uploader-ksu-uiconf-and-filetype-filters) for some useful defaults.  If you're self-hosted you probably know how to (or hopefully can figure out how to) set up your own ui_confs.
 * Restart both Apache and the automated jobs daemon (`sudo /etc/init.d/apache2 restart && sudo /etc/init.d/canvas_init restart`).

For more detailed information about setting up Kaltura, please see our [[Kaltura setup instructions]].

### Linked In
 * [Set up a developer accounts with LinkedIn](https://www.linkedin.com/secure/developer) and get your developer API key and secret key.  You'll need to fill out the details of your integration.  You don't need to enter an *OAuth Redirect URL*.
 * Copy `config/linked_in.yml.example` to `config/linked_in.yml` and open it up.  Find the *production* section of the configuration file and correct the configuration file to have your API key and API secret.
 * Restart both Apache and the automated jobs daemon (`sudo /etc/init.d/apache2 restart && sudo /etc/init.d/canvas_init restart`).

### Scribd

 * [Set up a developer account with Scribd](http://www.scribd.com/developers/signup_api) and [get your developer API key](http://www.scribd.com/account/edit#api).
 * Copy `config/scribd.yml.example` to `config/scribd.yml` and open it up. Find the *production* section of the configuration file and correct the configuration file to have your API key and API secret.
 * Restart both Apache and the automated jobs daemon (`sudo /etc/init.d/apache2 restart && sudo /etc/init.d/canvas_init restart`).

### TinyChat

 * Set up an account with TinyChat and [get a developer API key](http://tinychat.com/developer/dashboard/).
 * Once you have an API key, copy `config/tinychat.yml.example` to `config/tinychat.yml` and open it up. Find the *production* section of the configuration file and replace `api_key: key` with `api_key: <yourkey>`.
 * Restart both Apache and the automated jobs daemon (`sudo /etc/init.d/apache2 restart && sudo /etc/init.d/canvas_init restart`).

### Twitter

 * [Register your instance of Canvas with Twitter](http://dev.twitter.com/apps/new).
 * You'll need to specify the *Registered OAuth Callback URL* as `http://<your-canvas-domain>/oauth_success?service=twitter`.
 * Make a note of your *consumer key* and *consumer secret*.
 * Once you have these keys, copy `config/twitter.yml.example` to `config/twitter.yml` and open it up.  Find the *production* section of the configuration file and replace `api_key: key` with `api_key: <your_consumer_key>` and `secret_key: shared_secret` with `secret_key: <your_consumer_secret>`.
 * Restart both Apache and the automated jobs daemon (`sudo /etc/init.d/apache2 restart && sudo /etc/init.d/canvas_init restart`).
