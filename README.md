# ShareXen

ShareXen is an API, which is really just another ShareX custom uploader PHP script, but done right. It requires at least PHP 7.0 to run, along with the cURL extension if you plan to use Discord webhooks to log requests. No database is required.

This API returns strict JSON results. You can easily parse its answers within your own scripts.

If you have any problem setting this up, you can ask for help on [ShareX's Discord server](https://discordapp.com/invite/sharex) directly.

## Features

* File uploads (who would've guessed)
* File rename / deletion
* Secure deletion URLs
* Multi-user support
* Permission system
* Discord webhooks

Please note that some features (such as renaming a file) can't be used from ShareX directly, and are solely meant to be called by other scripts or programs. If you need ideas, go integrate this API into a Discord bot, which could ease management in case you plan to host it for quite a few users.

## Installing the API

1. Download the `sharexen.php` file from this repository.
2. Open it with a text editor, and edit the configuration to your needs.
3. Upload it to your webhost by any means, using whichever filename you want.

Files will be uploaded next to the script, in the same folder.

## Using the API with ShareX

Download the `ShareXen.sxcu` file from this repository and double-click it.  
If you can't open it for whatever reason, follow that procedure:

1. Open ShareX, click on `Destinations` then `Custom uploader settings…`.
2. Click the `Import` dropdown menu, and import the uploader file.

You will need to edit the `Request URL` field to match your own domain name and script path, along with the `token` argument value that must contain a valid user token previously defined within the API itself.

## Parameters and results

The form shall be encoded as `multipart/form-data`, with the file form name of `image` when calling the `upload` endpoint. The file must therefore be in binary format, not base64-encoded.

The following string parameters are recognized by the API:

| Name       | Request     | Description                                            |
| ---------- | ----------- | ------------------------------------------------------ |
| `token`    | POST only   | Authenticate for accessing restricted endpoints.       |
| `endpoint` | GET or POST | Specify which API endpoint you're requesting.          |
| `key`      | GET or POST | Secret security key for deleting / renaming a file.    |
| `filename` | GET or POST | For endpoints supporting / requiring a filename.       |
| `new_name` | GET or POST | For the `rename` endpoint, file mustn't already exist. |
| `domain`   | GET or POST | Set a custom domain name for the `url` answer field.   |
| `protocol` | GET or POST | Set a custom protocol for the `url` answer field.      |

The following endpoints are supported:

| Name     | Supported parameters                                                |
| -------- | ------------------------------------------------------------------- |
| `info`   | `token` (user), `filename`                                          |
| `upload` | `token` (user), `image` (file), `filename`                          |
| `delete` | `token` (admin) or `key`, `filename`                                |
| `rename` | `token` (admin) or `token` (user) and `key`, `filename`, `new_name` |

Using the `filename` parameter for the `upload` endpoint and accessing the `rename` parameter can be restricted by the configuration. Refer to the available options for more information.

The following JSON fields can be returned:

| Name              | Endpoints           | Type    | Description                                      |
| ----------------- | ------------------- | ------- | ------------------------------------------------ |
| `api_version`     | all                 | String  | Current API version number (SemVer)              |
| `api_source`      | all                 | String  | URL to the GitHub source repository              |
| `endpoint`        | all                 | String  | Called API endpoint, or `unknown`                |
| `username`        | all                 | String  | Username, only for authenticated requests        |
| `status`          | all                 | String  | Request status, either `success` or `error`      |
| `http_code`       | all                 | Integer | Mirror of the returned HTTP code                 |
| `filename`        | all                 | String  | Name of the file as stored on the server         |
| `execution_time`  | all                 | Float   | Script execution time, in seconds                |
| `iteration_count` | `upload`            | Integer | Attempts at generating a unique filename         |
| `url`             | `upload` & `rename` | String  | URL for the new file                             |
| `key`             | `upload` & `rename` | String  | Security key for the new file                    |
| `deletion_url`    | `upload` & `rename` | String  | Full deletion URL for the new file               |
| `method`          | `delete` & `rename` | String  | Authentication method used to call the endpoint  |
| `old_name`        | `rename`            | String  | Previous name of the file                        |
| `error`           | any                 | String  | Static error code, only sent if anything fails   |
| `error_msg`       | any                 | String  | Human-readable error, sent with all error codes  |
| `debug`           | any                 | String  | Human-readable information, only for some errors |

The `info` endpoint implements several JSON fields, which can be returned or not depending on your access level, whether you specify a filename, and whether it exists. Here is the full specification:

| Name                 | Admin | Filename    | Type             | Description                                                          |
| -------------------- | ----- | ----------- | ---------------- | -------------------------------------------------------------------- |
| `is_admin`           | No    | Irrelevant  | Boolean          | Admin status of the caller                                           |
| `file_exists`        | No    | Specified   | Boolean          | Whether the specified file exists or not                             |
| `filename`           | No    | Specified   | String           | (File must exist) Name of the file                                   |
| `filesize`           | No    | Specified   | Integer          | (File must exist) Size of the file in bytes                          |
| `uploaded_at`        | No    | Specified   | Integer          | (File must exist) File upload timestamp                              |
| `url`                | No    | Specified   | String           | (File must exist) URL to the file                                    |
| `key`                | Yes   | Specified   | String           | (File must exist) Security key of the file                           |
| `deletion_url`       | Yes   | Specified   | String           | (File must exist) Deletion URL of the file                           |
| `endpoints`          | No    | Unspecified | Array of Strings | List of supported API endpoints                                      |
| `keyspace`           | No    | Unspecified | String           | Keyspace used by the API (configuration)                             |
| `name_length`        | No    | Unspecified | Integer          | Size of random names (configuration)                                 |
| `max_iterations`     | No    | Unspecified | Integer          | Max amount of attempts at generating a unique name (configuration)   |
| `allowed_extensions` | No    | Unspecified | Array of Strings | List of allowed file extensions (configuration)                      |
| `allowed_characters` | No    | Unspecified | String           | Additional allowed characters, for custom filenames (configuration)  |
| `custom_names`       | No    | Unspecified | Boolean          | Whether custom filenames are globally allowed or not (configuration) |
| `files_count`        | No    | Unspecified | Integer          | Amount of files (matching allowed extensions) in the current folder  |
| `files`              | Yes   | Unspecified | Array of Strings | List of files (matching allowed extensions) in the current folder    |
| `users`              | Yes   | Unspecified | Array of Strings | List of configured users, usernames only (configuration)             |
| `admins`             | Yes   | Unspecified | Array of Strings | List of instance administrators (configuration)                      |
| `can_use_webhook`    | Yes   | Unspecified | Boolean          | Whether PHP's `allow_url_fopen` configuration option is enabled      |
| `discord_webhook`    | Yes   | Unspecified | Boolean          | Whether a Discord webhook has been set up or not (configuration)     |

## Limitations and security

As this script doesn't use any database, there isn't any feature such as ratelimiting, so you'll have to handle that using your webserver itself, or an intermediate service like Cloudflare.

If enabled, the Discord webhook can be called for each API call depending on your configuration. If you receive a lot of requests, you might hit the webhook ratelimit, which cannot be handled by this script and will therefore be ignored.

As a security measure, this script doesn't accept files that aren't recognized as images or videos, based on their mime type. That can be modified, but here again, do it at your own risk, I won't support you there.

Security keys are only as secure as your `SALT` is. Make sure to have a **very random** string there containing basically any character you want, and of course **never** share it with anyone. I can't stress this enough, as it would allow anyone having it to compute the security key for any image file. Keep in mind that having to change the deletion salt means all previously generated keys will be rendered invalid. Of course, take great care of user tokens too, especially admin ones (if you have any) since they can be more destructive than security keys, although you can safely update any of them without breaking any key whatsoever.

## Contributing

If you want to help improve this API or its documentation, please join the [ShareX Discord server](https://discord.com/invite/sharex) and mention `Xenthys#0001` in the `#programming` channel to suggest modifications. You can also open a pull-request, but make sure to **sign your commits with GPG** and **respect the script's programming style** so we can keep it as readable as possible, thank you.
