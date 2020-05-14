Canvas has a series of advanced settings that can be customized to tweak behavior of the app. The way this system works is that the app requests a setting key and provides a default, and the default is used unless a value is found in the database. So, for example, to set a different code cogs base url, you would run:

```
Setting.set('codecogs.equation_image_link', "<your custom url>")
```

in a rails console, and then when the app runs code to get that setting:

```
Setting.get('codecogs.equation_image_link', 'http://latex.codecogs.com/gif.latex')
```

even though it still specifies a default, your customization is returned instead.

Generally we try to use these anywhere that we would have a constant in the code that we might want to tweak. We try to use sane defaults so that you shouldn't need to change any of these for a standard working version of canvas.

We don't currently have good documentation around these settings, but here is a script-generated list of settings, and how they are used. (https://gist.github.com/simonista/5a6b27c1a03a0da53acf)

#### block_html_frames

- default: `'true'`
- usage: `if !files_domain? && Setting.get('block_html_frames', 'true') == 'true' && !@embeddable`
- location: `app/controllers/application_controller.rb:221`

#### calendar_feed_previous_days

- default: `'30'`
- usage: `@start_date = Setting.get('calendar_feed_previous_days', '30').to_i.days.ago`
- location: `app/controllers/calendar_events_api_controller.rb:521`

#### calendar_feed_upcoming_days

- default: `'366'`
- usage: `@end_date = Setting.get('calendar_feed_upcoming_days', '366').to_i.days.from_now`
- location: `app/controllers/calendar_events_api_controller.rb:522`

#### codecogs.equation_image_link

- default: `'http://latex.codecogs.com/gif.latex'`
- usage: `base_url = Setting.get('codecogs.equation_image_link', 'http://latex.codecogs.com/gif.latex')`
- location: `app/controllers/equation_images_controller.rb:8`

#### error_search_enabled

- default: `"true"`
- usage: `Setting.get("error_search_enabled", "true") == "true"`
- location: `app/controllers/errors_controller.rb:50`

#### attachment_json_response_max_size

- default: `1.megabyte.to_s`
- usage: `if @attachment.size > Setting.get('attachment_json_response_max_size', 1.megabyte.to_s).to_i`
- location: `app/controllers/files_controller.rb:590`

#### api_max_per_page

- default: `'50'`
- usage: `per_page = Setting.get('api_max_per_page', '50').to_i`
- location: `app/controllers/gradebooks_controller.rb:179`

#### gradebook2.submissions_chunk_size

- default: `'35'`
- usage: `:chunk_size => Setting.get('gradebook2.submissions_chunk_size', '35').to_i,`
- location: `app/controllers/gradebooks_controller.rb:184`

#### sis_app_url

- default: `nil`
- usage: `:sis_app_url => Setting.get('sis_app_url', nil),`
- location: `app/controllers/gradebooks_controller.rb:218`

#### sis_app_token

- default: `nil`
- usage: `:sis_app_token => Setting.get('sis_app_token', nil)`
- location: `app/controllers/gradebooks_controller.rb:219`

#### gradebook_history_submission_count_threshold

- default: `'0'`
- usage: `submissions_limit = Setting.get('gradebook_history_submission_count_threshold', '0').to_i`
- location: `app/controllers/gradebooks_controller.rb:232`

#### running_jobs_refresh_seconds

- default: `2.seconds.to_s`
- usage: `@running_jobs_refresh_seconds = Setting.get('running_jobs_refresh_seconds', 2.seconds.to_s).to_f`
- location: `app/controllers/jobs_controller.rb:22`

#### job_tags_refresh_seconds

- default: `10.seconds.to_s`
- usage: `@job_tags_refresh_seconds = Setting.get('job_tags_refresh_seconds', 10.seconds.to_s).to_f`
- location: `app/controllers/jobs_controller.rb:23`

#### oauth.allowed_timestamp_delta

- default: `90.minutes.to_s`
- usage: `allowed_delta = Setting.get('oauth.allowed_timestamp_delta', 90.minutes.to_s).to_i`
- location: `app/controllers/lti_api_controller.rb:92`

#### page_views_csv_export_rows

- default: `'300'`
- usage: `per_page = Setting.get('page_views_csv_export_rows', '300').to_i`
- location: `app/controllers/page_views_controller.rb:198`

#### google_analytics_key

- default: `nil`
- usage: `:GOOGLE_ANALYTICS_KEY => Setting.get('google_analytics_key', nil),`
- location: `app/controllers/pseudonym_sessions_controller.rb:129`

#### google_analytics_key

- default: `nil`
- usage: `js_env :GOOGLE_ANALYTICS_KEY => Setting.get('google_analytics_key', nil)`
- location: `app/controllers/pseudonym_sessions_controller.rb:692`

#### skip_sis_jobs_account_ids

- default: `'').split(',').include?(@account.global_id.to_s`
- usage: `unless Setting.get('skip_sis_jobs_account_ids', '').split(',').include?(@account.global_id.to_s)`
- location: `app/controllers/sis_imports_api_controller.rb:330`

#### google_analytics_key

- default: `nil`
- usage: `:googleAnalyticsAccount => Setting.get('google_analytics_key', nil),`
- location: `app/helpers/application_helper.rb:452`

#### show_feedback_link

- default: `"false"`
- usage: `show_feedback_link = Setting.get("show_feedback_link", "false") == "true"`
- location: `app/helpers/application_helper.rb:656`

#### access_token_last_used_threshold

- default: `10.minutes`
- usage: `Setting.get('access_token_last_used_threshold', 10.minutes).to_i`
- location: `app/models/access_token.rb:48`

#### account_notification_default_months_in_display_cycle

- default: `"9"`
- usage: `Setting.get("account_notification_default_months_in_display_cycle", "9").to_i`
- location: `app/models/account_notification.rb:100`

#### aac_debug_expire_minutes

- default: `30`
- usage: `Setting.get('aac_debug_expire_minutes', 30).minutes`
- location: `app/models/account_authorization_config.rb:372`

#### ldap_failure_wait_time

- default: `1.minute.to_s`
- usage: `Setting.get('ldap_failure_wait_time', 1.minute.to_s).to_i`
- location: `app/models/account_authorization_config.rb:376`

#### ldap_timelimit

- default: `5.seconds.to_s`
- usage: `default_timeout = Setting.get('ldap_timelimit', 5.seconds.to_s).to_f`
- location: `app/models/account_authorization_config.rb:382`

#### terms_of_use_url

- default: `'http://www.canvaslms.com/policies/terms-of-use'`
- usage: `Setting.get('terms_of_use_url', 'http://www.canvaslms.com/policies/terms-of-use')`
- location: `app/models/account.rb:285`

#### privacy_policy_url

- default: `'http://www.canvaslms.com/policies/privacy-policy'`
- usage: `Setting.get('privacy_policy_url', 'http://www.canvaslms.com/policies/privacy-policy')`
- location: `app/models/account.rb:289`

#### terms_required

- default: `'true'`
- usage: `Setting.get('terms_required', 'true') == 'true'`
- location: `app/models/account.rb:293`

#### account_default_quota

- default: `500.megabytes.to_s`
- usage: `Setting.get('account_default_quota', 500.megabytes.to_s).to_i`
- location: `app/models/account.rb:495`

#### account_default_quota

- default: `500.megabytes.to_s`
- usage: `Setting.get('account_default_quota', 500.megabytes.to_s).to_i`
- location: `app/models/account.rb:502`

#### #{special_account_type}\_account_id

- default: `nil`
- usage: `special_account_id = @special_account_ids[special_account_type] ||= Setting.get("#{special_account_type}_account_id", nil)`
- location: `app/models/account.rb:937`

#### #{special_account_type}\_account_id

- default: `nil`
- usage: `special_account_id = Setting.get("#{special_account_type}_account_id", nil)`
- location: `app/models/account.rb:942`

#### too_many_quiz_submission_versions

- default: `"150"`
- usage: `qs_threshold = Setting.get("too_many_quiz_submission_versions", "150").to_i`
- location: `app/models/assignment.rb:1352`

#### attachment_build_media_object_delay_seconds

- default: `10.to_s`
- usage: `delay = Setting.get('attachment_build_media_object_delay_seconds', 10.to_s).to_i`
- location: `app/models/attachment.rb:296`

#### attachment_minimum_size_for_quota

- default: `'512'`
- usage: `Setting.get('attachment_minimum_size_for_quota', '512').to_i`
- location: `app/models/attachment.rb:675`

#### context_default_quota

- default: `50.megabytes.to_s`
- usage: `quota = Setting.get('context_default_quota', 50.megabytes.to_s).to_i`
- location: `app/models/attachment.rb:684`

#### attachment_notify_quiet_period_minutes

- default: `"5"`
- usage: `quiet_period = Setting.get("attachment_notify_quiet_period_minutes", "5").to_i.minutes.ago`
- location: `app/models/attachment.rb:874`

#### attachment_notify_discard_older_than_hours

- default: `"120"`
- usage: `discard_older_than = Setting.get("attachment_notify_discard_older_than_hours", "120").to_i.hours.ago`
- location: `app/models/attachment.rb:877`

#### filter_scribd_submits

- default: `"false"`
- usage: `Setting.get("filter_scribd_submits", "false") == "true"`
- location: `app/models/attachment.rb:1209`

#### max_canvadocs_attempts

- default: `'5'`
- usage: `if attempt <= Setting.get('max_canvadocs_attempts', '5').to_i`
- location: `app/models/attachment.rb:1436`

#### max_crocodoc_attempts

- default: `'5'`
- usage: `if attempt <= Setting.get('max_crocodoc_attempts', '5').to_i`
- location: `app/models/attachment.rb:1456`

#### scribd.stale_threshold

- default: `120`
- usage: `cutoff = Setting.get('scribd.stale_threshold', 120).to_f.days.ago`
- location: `app/models/attachment.rb:1666`

#### attachment_thumbnail_sizes

- default: `"").split(","`
- usage: `DYNAMIC_THUMBNAIL_SIZES + Setting.get("attachment_thumbnail_sizes", "").split(",")`
- location: `app/models/attachment.rb:1702`

#### content_participation_count_ttl

- default: `30.minutes`
- usage: `Setting.get('content_participation_count_ttl', 30.minutes).to_i`
- location: `app/models/content_participation_count.rb:124`

#### populate_conversation_participants_private_hash_complete

- default: `'0'`
- usage: `if Setting.get('populate_conversation_participants_private_hash_complete', '0') == '0'`
- location: `app/models/conversation.rb:91`

#### crocodoc_status_check_batch_size

- default: `'45'`
- usage: `bs = Setting.get('crocodoc_status_check_batch_size', '45').to_i`
- location: `app/models/crocodoc_document.rb:129`

#### course_default_quota

- default: `500.megabytes.to_s`
- usage: `Setting.get('course_default_quota', 500.megabytes.to_s).to_i`
- location: `app/models/course.rb:1001`

#### #{default_key_name}\_developer_key_id

- default: `nil)`
- usage: `if (key_id = Setting.get("#{default_key_name}_developer_key_id", nil)) && key_id.present?`
- location: `app/models/developer_key.rb:63`

#### discussion_entry_max_depth

- default: `'50'`
- usage: `Setting.get('discussion_entry_max_depth', '50').to_i`
- location: `app/models/discussion_entry.rb:109`

#### recompute_grades_window

- default: `600).to_i.seconds`
- usage: `Rails.cache.fetch(['recompute_final_scores', course.id, user].cache_key, :expires_in => Setting.get('recompute_grades_window', 600).to_i.seconds) do`
- location: `app/models/enrollment.rb:784`

#### enrollment_last_activity_at_threshold

- default: `2.minutes`
- usage: `Setting.get('enrollment_last_activity_at_threshold', 2.minutes).to_i`
- location: `app/models/enrollment.rb:1020`

#### enrollment_total_activity_time_threshold

- default: `10.minutes`
- usage: `Setting.get('enrollment_total_activity_time_threshold', 10.minutes).to_i`
- location: `app/models/enrollment.rb:1024`

#### group_default_quota

- default: `50.megabytes.to_s`
- usage: `Setting.get('group_default_quota', 50.megabytes.to_s).to_i`
- location: `app/models/group.rb:531`

#### max_groups_in_new_category

- default: `'200'`
- usage: `[num, Setting.get('max_groups_in_new_category', '200').to_i].min :`
- location: `app/models/group_category.rb:371`

#### media_bulk_upload_timeout

- default: `30.minutes.to_s`
- usage: `timeout = Setting.get('media_bulk_upload_timeout', 30.minutes.to_s).to_i`
- location: `app/models/kaltura_media_file_handler.rb:71`

#### media_object_bulk_refresh_max_attempts

- default: `'5'`
- usage: `if attempt < Setting.get('media_object_bulk_refresh_max_attempts', '5').to_i`
- location: `app/models/media_object.rb:145`

#### media_object_bulk_refresh_wait_period

- default: `'30'`
- usage: `wait_period = Setting.get('media_object_bulk_refresh_wait_period', '30').to_i`
- location: `app/models/media_object.rb:146`

#### enable_page_views

- default: `'false'`
- usage: `enable_page_views = Setting.get('enable_page_views', 'false')`
- location: `app/models/page_view.rb:109`

#### page_views_scrollback_limit:users

- default: `52.weeks`
- usage: `scrollback_limit -> { Setting.get('page_views_scrollback_limit:users', 52.weeks) }`
- location: `app/models/page_view.rb:144`

#### skip_pageview_updates

- default: `'false'`
- usage: `Setting.get('skip_pageview_updates', 'false') != 'true'`
- location: `app/models/page_view.rb:189`

#### page_views_store_active_user_counts

- default: `'false'`
- usage: `return unless Setting.get('page_views_store_active_user_counts', 'false') == 'redis' && Canvas.redis_enabled?`
- location: `app/models/page_view.rb:304`

#### page_views_active_user_exptime

- default: `1.day.to_s`
- usage: `exptime = Setting.get('page_views_active_user_exptime', 1.day.to_s).to_i`
- location: `app/models/page_view.rb:306`

#### quiz_statistics_max_questions

- default: `DefaultMaxQuestions).to_i`
- usage: `Setting.get('quiz_statistics_max_questions', DefaultMaxQuestions).to_i) ||`
- location: `app/models/quizzes/quiz_statistics.rb:50`

#### quiz_statistics_max_submissions

- default: `DefaultMaxSubmissions).to_i`
- usage: `Setting.get('quiz_statistics_max_submissions', DefaultMaxSubmissions).to_i)`
- location: `app/models/quizzes/quiz_statistics.rb:52`

#### usage_statistics_collection

- default: `"opt_out"`
- usage: `collection_type = Setting.get("usage_statistics_collection", "opt_out")`
- location: `app/models/report_snapshot.rb:85`

#### installation_uuid

- default: `""`
- usage: `installation_uuid = Setting.get("installation_uuid", "")`
- location: `app/models/report_snapshot.rb:88`

#### sis_batch_process_start_delay

- default: `'0'`
- usage: `process_delay = Setting.get('sis_batch_process_start_delay', '0').to_f`
- location: `app/models/sis_batch.rb:97`

#### sis_batch_max_messages

- default: `'1000'`
- usage: `Setting.get('sis_batch_max_messages', '1000').to_i`
- location: `app/models/sis_batch.rb:259`

#### stream_items_ttl

- default: `4.weeks`
- usage: `ttl = Setting.get('stream_items_ttl', 4.weeks).to_i.ago`
- location: `app/models/stream_item.rb:318`

#### turnitin_submission_delay_seconds

- default: `60.to_s`
- usage: `delay = Setting.get('turnitin_submission_delay_seconds', 60.to_s).to_i`
- location: `app/models/submission.rb:336`

#### max_messages_per_day_per_user

- default: `500`
- usage: `Setting.get('max_messages_per_day_per_user', 500).to_i`
- location: `app/models/user.rb:1104`

#### recent_stream_item_limit

- default: `100)`
- usage: `limit(Setting.get('recent_stream_item_limit', 100))`
- location: `app/models/user.rb:1801`

#### min_users_for_conversation_context_codes_preload

- default: `5`
- usage: `return if users.length < Setting.get("min_users_for_conversation_context_codes_preload", 5).to_i`
- location: `app/models/user.rb:1969`

#### user_default_quota

- default: `50.megabytes.to_s`
- usage: `Setting.get('user_default_quota', 50.megabytes.to_s).to_i`
- location: `app/models/user.rb:2073`

#### account_authorization_config_ip_addresses

- default: `nil`
- usage: `<% if ips = Setting.get('account_authorization_config_ip_addresses', nil).presence %>`
- location: `app/views/account_authorization_configs/index.html.erb:40`

#### show_feedback_link

- default: `"false"`
- usage: `<% if @account.root_account? && !@account.site_admin? && Setting.get("show_feedback_link", "false") == "true" %>`
- location: `app/views/accounts/settings.html.erb:366`

#### api_max_per_page

- default: `'50')`
- usage: `js_env :SYLLABUS_PER_PAGE => Setting.get('syllabus_per_page', Setting.get('api_max_per_page', '50'))`
- location: `app/views/assignments/syllabus.html.erb:7`

#### show_feedback_link

- default: `"false"`
- usage: `<% if Setting.get("show_feedback_link", "false") == "true" %>`
- location: `app/views/communication_channels/confirm_failed.html.erb:16`

#### jobs.slow_threshold

- default: `600`
- usage: `:slow_threshold => Setting.get('jobs.slow_threshold', 600).to_f,`
- location: `app/views/jobs/index.html.erb:162`

#### jobs.super_slow_threshold

- default: `3600`
- usage: `:super_slow_threshold => Setting.get('jobs.super_slow_threshold', 3600).to_f`
- location: `app/views/jobs/index.html.erb:163`

#### show_opensource_linkback

- default: `"false"`
- usage: `<% if Setting.get("show_opensource_linkback", "false") == "true" %>`
- location: `app/views/layouts/application.html.erb:182`

#### registration_link

- default: `"/register_from_website"`
- usage: `<%= link_to t("register_for_canvas", "*Need a Canvas Account?* **Click Here, It's Free!**", :wrapper => {'*' => '<i>\1</i>', '**' => '<b>\1</b>'}), Setting.get("registration_link", "/register_from_website"), :id => 'register_link', :class => 'not_external register_banner' %>`
- location: `app/views/shared/_login.html.erb:33`

#### show_feedback_link

- default: `"false"`
- usage: `<% if Setting.get("show_feedback_link", "false") == "true" %>`
- location: `app/views/shared/unauthorized.html.erb:47`

#### session_secret_key

- default: `SecureRandom.hex(64)) rescue SecureRandom.hex(64)`
- usage: `:secret => (Setting.get_or_set("session_secret_key", SecureRandom.hex(64)) rescue SecureRandom.hex(64))`
- location: `config/initializers/session_store.rb:10`

#### error_reports_retain_for

- default: `3.months.to_s`
- usage: `cutoff = Setting.get('error_reports_retain_for', 3.months.to_s).to_i`
- location: `config/periodic_jobs.rb:82`

#### respondus_endpoint.polling_api

- default: `'true'`
- usage: `if Setting.get('respondus_endpoint.polling_api', 'true') != 'false'`
- location: `gems/plugins/respondus_soap_endpoint/lib/respondus_soap_endpoint/urn:RespondusAPIServant.rb:558`

#### respondus_endpoint.polling_time

- default: `'2').to_f`
- usage: `sleep(Setting.get('respondus_endpoint.polling_time', '2').to_f)`
- location: `gems/plugins/respondus_soap_endpoint/lib/respondus_soap_endpoint/urn:RespondusAPIServant.rb:567`

#### api_max_per_page

- default: `'50'`
- usage: `if includes.include?('items') && count <= Setting.get('api_max_per_page', '50').to_i`
- location: `lib/api/v1/context_module.rb:44`

#### api_max_per_page

- default: `'50'`
- usage: `Setting.get('api_max_per_page', '50').to_i`
- location: `lib/api.rb:227`

#### api_per_page

- default: `'10'`
- usage: `per_page = controller.params[:per_page] || options[:default] || Setting.get('api_per_page', '10')`
- location: `lib/api.rb:231`

#### oath_token_request_timeout

- default: `10.minutes.to_s).to_i, code_data.to_json`
- usage: `Canvas.redis.setex("#{REDIS_PREFIX}#{code}", Setting.get('oath_token_request_timeout', 10.minutes.to_s).to_i, code_data.to_json)`
- location: `lib/canvas/oauth/token.rb:95`

#### ignore_redis_failures

- default: `'true'`
- usage: `Setting.get('ignore_redis_failures', 'true') == 'true'`
- location: `lib/canvas/redis.rb:29`

#### redis_failure_time

- default: `'300').to_i rescue 300`
- usage: `return (Time.now - last_redis_failure[redis_name]) < (Setting.get('redis_failure_time', '300').to_i rescue 300)`
- location: `lib/canvas/redis.rb:35`

#### request_throttle.enabled

- default: `"true"`
- usage: `if Setting.get("request_throttle.enabled", "true") == "true"`
- location: `lib/canvas/request_throttle.rb:88`

#### request_throttle.#{setting}

- default: `default`
- usage: `Setting.get("request_throttle.#{setting}", default).to_f`
- location: `lib/canvas/request_throttle.rb:215`

#### encryption_key_hash

- default: `nil`
- usage: `db_hash = Setting.get('encryption_key_hash', nil) rescue return # in places like rake db:test:reset, we don't care that the db/table doesn't exist`
- location: `lib/canvas/security.rb:67`

#### login_attempts_total

- default: `'20'`
- usage: `total_allowed = Setting.get('login_attempts_total', '20').to_i`
- location: `lib/canvas/security.rb:95`

#### login_attempts_per_ip

- default: `'10'`
- usage: `ip_allowed = Setting.get('login_attempts_per_ip', '10').to_i`
- location: `lib/canvas/security.rb:96`

#### login_attempts_ttl

- default: `5.minutes.to_s`
- usage: `exptime = Setting.get('login_attempts_ttl', 5.minutes.to_s).to_i`
- location: `lib/canvas/security.rb:112`

#### service_generic_timeout

- default: `15.seconds.to_s)`
- usage: `timeout = (Setting.get("service_#{service_name}_timeout", nil) || options[:fallback_timeout_length] || Setting.get("service_generic_timeout", 15.seconds.to_s)).to_f`
- location: `lib/canvas.rb:159`

#### service_generic_cutoff

- default: `3.to_s)`
- usage: `cutoff = (Setting.get("service_#{service_name}_cutoff", nil) || Setting.get("service_generic_cutoff", 3.to_s)).to_i`
- location: `lib/canvas.rb:163`

#### service_generic_error_ttl

- default: `1.minute.to_s)`
- usage: `error_ttl = (Setting.get("service_#{service_name}_error_ttl", nil) || Setting.get("service_generic_error_ttl", 1.minute.to_s)).to_i`
- location: `lib/canvas.rb:164`

#### exporter_media_object_flavor

- default: `nil).presence`
- usage: `:media_object_flavor => Setting.get('exporter_media_object_flavor', nil).presence)`
- location: `lib/cc/qti/qti_manifest.rb:34`

#### exporter_media_object_flavor

- default: `nil).presence`
- usage: `:media_object_flavor => Setting.get('exporter_media_object_flavor', nil).presence)`
- location: `lib/cc/resource.rb:48`

#### filter_page_view_url_params_batch_size

- default: `'1000'`
- usage: `Setting.get('filter_page_view_url_params_batch_size', '1000').to_i`
- location: `lib/data_fixup/filter_page_view_url_params.rb:41`

#### fix_audit_log_uuid_indexes_batch_size

- default: `1000`
- usage: `@batch_size ||= Setting.get('fix_audit_log_uuid_indexes_batch_size', 1000).to_i`
- location: `lib/data_fixup/fix_audit_log_uuid_indexes.rb:47`

#### media_recording_type_bad_date

- default: `'03/08/2013'), '%m/%d/%Y'`
- usage: `date = Date.strptime(Setting.get('media_recording_type_bad_date', '03/08/2013'), '%m/%d/%Y')`
- location: `lib/data_fixup/fix_media_recording_submission_types.rb:4`

#### support_multiple_account_types

- default: `"true"`
- usage: `if Setting.get("support_multiple_account_types", "true") == "true"`
- location: `lib/external_statuses.rb:28`

#### recently_logged_in_timespan

- default: `30.days.to_s`
- usage: `timespan = Setting.get('recently_logged_in_timespan', 30.days.to_s).to_i.seconds`
- location: `lib/reporting/counts_report.rb:68`

#### sis_batch_progress_interval

- default: `2.seconds`
- usage: `update_interval = Setting.get('sis_batch_progress_interval', 2.seconds).to_i`
- location: `lib/sis/csv/import.rb:239`

#### sis_batch_pause_every

- default: `100)`
- usage: `@pause_every = (@batch.data[:pause_every] || Setting.get('sis_batch_pause_every', 100)).to_i`
- location: `lib/sis/csv/import.rb:321`

#### sis_batch_pause_duration

- default: `0)`
- usage: `@pause_duration = (@batch.data[:pause_duration] || Setting.get('sis_batch_pause_duration', 0)).to_f`
- location: `lib/sis/csv/import.rb:322`

#### sis_transaction_seconds

- default: `'1'`
- usage: `transaction_timeout = Setting.get('sis_transaction_seconds', '1').to_i.seconds`
- location: `lib/sis/enrollment_importer.rb:105`

#### sis_transaction_seconds

- default: `'1'`
- usage: `transaction_timeout = Setting.get('sis_transaction_seconds', '1').to_i.seconds`
- location: `lib/sis/user_importer.rb:82`

#### summary_message_consolidator_batch_size

- default: `'500').to_i, false`
- usage: `dm_id_batches.in_groups_of(Setting.get('summary_message_consolidator_batch_size', '500').to_i, false) do |batches|`
- location: `lib/summary_message_consolidator.rb:36`

#### max_zip_file_count

- default: `'100000'`
- usage: `max = Setting.get('max_zip_file_count', '100000').to_i`
- location: `lib/unzip_attachment.rb:265`

#### user_search_with_gist

- default: `'true'`
- usage: `Setting.get('user_search_with_gist', 'true') == 'true'`
- location: `lib/user_search.rb:91`

#### user_search_with_full_complexity

- default: `'true'`
- usage: `Setting.get('user_search_with_full_complexity', 'true') == 'true'`
- location: `lib/user_search.rb:95`

#### api_max_per_page

- default: `'2') }`
- usage: `{ :controller => "page_views", :action => "index", :user_id => @student.to_param, :format => 'json', :page => page, :per_page => Setting.get('api_max_per_page', '2') })`
- location: `spec/apis/v1/users_api_spec.rb:291`

#### max_zip_file_count

- default: `'100000'`
- usage: `current_setting = Setting.get('max_zip_file_count', '100000')`
- location: `spec/lib/unzip_attachment_spec.rb:138`

#### rspec_developer_key_id

- default: `nil`
- usage: `key_id = Setting.get('rspec_developer_key_id', nil)`
- location: `spec/models/developer_key_spec.rb:40`

#### crocodoc_counter

- default: `0`
- usage: `Setting.get('crocodoc_counter', 0).to_i.should eql 998`
- location: `spec/models/user_spec.rb:2228`

#### academic_benchmark_migration_user_id

- default: `nil`
- usage: `user_id = Setting.get("academic_benchmark_migration_user_id", nil)`
- location: `vendor/plugins/academic_benchmark/lib/academic_benchmark.rb:18`

#### #{full_strand_name}\_num_strands

- default: `nil`
- usage: `num_strands = Setting.get("#{full_strand_name}_num_strands", nil)`
- location: `vendor/plugins/delayed_job/lib/delayed/backend/base.rb:61`

#### #{strand_name}\_num_strands

- default: `nil`
- usage: `num_strands ||= Setting.get("#{strand_name}_num_strands", nil)`
- location: `vendor/plugins/delayed_job/lib/delayed/backend/base.rb:66`

#### jobs_get_next_batch_size

- default: `'5'`
- usage: `self.batch_size ||= Setting.get('jobs_get_next_batch_size', '5').to_i`
- location: `vendor/plugins/delayed_job/lib/delayed/backend/active_record.rb:207`

#### jobs_select_random

- default: `'false'`
- usage: `self.select_random = Setting.get('jobs_select_random', 'false') == 'true'`
- location: `vendor/plugins/delayed_job/lib/delayed/backend/active_record.rb:209`

#### delayed_jobs_stats_ttl

- default: `1.month.to_s`
- usage: `ttl = Setting.get('delayed_jobs_stats_ttl', 1.month.to_s).to_i.from_now`
- location: `vendor/plugins/delayed_job/lib/delayed/stats.rb:29`

#### delayed_jobs_store_stats

- default: `'false'`
- usage: `Setting.get('delayed_jobs_store_stats', 'false') == 'redis'`
- location: `vendor/plugins/delayed_job/lib/delayed/stats.rb:36`

#### delayed_jobs_sleep_delay

- default: `'5.0'`
- usage: `@sleep_delay ||= Setting.get('delayed_jobs_sleep_delay', '5.0').to_f`
- location: `vendor/plugins/delayed_job/lib/delayed/worker.rb:105`

#### delayed_jobs_sleep_delay_stagger

- default: `'2.5'`
- usage: `@sleep_delay_stagger ||= Setting.get('delayed_jobs_sleep_delay_stagger', '2.5').to_f`
- location: `vendor/plugins/delayed_job/lib/delayed/worker.rb:106`

#### delayed_jobs_unique_tmpdir

- default: `'true'`
- usage: `@make_tmpdir ||= Setting.get('delayed_jobs_unique_tmpdir', 'true') == 'true'`
- location: `vendor/plugins/delayed_job/lib/delayed/worker.rb:107`
