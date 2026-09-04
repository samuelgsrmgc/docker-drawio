This docker compose file runs draw.io diagram editor without depending on any draw.io online services (e.g., the export service) and with support of Google Drive, Microsoft OneDrive, and GitLab.

### Adding fonts to improve generated PDFs and images

The docker-compose file bind the `fonts` volume into the running container system fonts.

These fonts are used by the export server (the `image-export` service, where the volume is mounted) for server-side rendering of exported PDFs/images only — they do not appear in the editor's font list. To make fonts selectable in the editor, see [Custom fonts](../README.md#custom-fonts) in the main README.

The best option for Windows users is to copy the contents of `Windowsdrive:/Windows/Fonts` into `fonts` folder. These fonts are copyrighted and cannot be re-distributed freely.

# Configuration

You can customize the application by setting the following environment variables.

* `DRAWIO_SERVER_URL`: Your deployment URL with a trailing slash. For example, `https://drawio.example.com/`, or `https://www.example.com/drawio/` if deployed into a sub-path. Setting this alone is enough — `DRAWIO_BASE_URL` is derived from it by stripping the trailing slash. The entrypoint also uses the sub-path (if any) to update the Tomcat context path automatically.
* `DRAWIO_BASE_URL`: (Optional, backwards-compat) Same URL without the trailing slash, used by the viewer/lightbox/embed code paths. Only set this if you don't set `DRAWIO_SERVER_URL`; the entrypoint will derive `DRAWIO_SERVER_URL` from it (by adding a trailing slash). If both are set, both are used as given.
* `DRAWIO_CSP_HEADER`: (Optional) Your website Content-Security-Policy if you want to customize it.
* `DRAWIO_VIEWER_URL`: (Optional) If you want to host a draw.io viewer also, set the viewer URL. For example, `https://drawio.example.com/js/viewer.min.js`
* `DRAWIO_LIGHTBOX_URL`: (Optional) If you want to host a draw.io viewer also, set the lightbox URL. For example, `https://drawio.example.com` 
* `DRAWIO_CONFIG`: (Optional) draw.io configuration JSON. [Documentation](https://www.drawio.com/doc/faq/configure-diagram-editor)
* `DRAWIO_CONFIG_FILE`: (Optional) Path inside the container of a file holding the configuration JSON instead of `DRAWIO_CONFIG`; add a volume for it. See the [main README](../README.md#editor-configuration).
* `DRAWIO_LANG`: (Optional) Default UI language code, e.g. `es`. See the [main README](../README.md#editor-configuration).
* `DRAWIO_USE_HTTP`: (Optional and INSECURE) If your setup uses http only and you understand the risks (for example, sending OAuth tokens over http), set `DRAWIO_USE_HTTP=1`. **Caution: Use at your own risk**.

## Google Drive

You will need to create a project at [Google API Console](https://console.developers.google.com/apis) and create [Credentials](https://console.developers.google.com/apis/credentials) of type "Create OAuth client ID" -> Web Application. This option will be disabled until you create "OAuth consent screen" from the link in warning message bar. In the "OAuth consent screen" configuration, enter the "Application name" and "Authorized domains". In the "Create OAuth client ID" configuration, enter the required information and note that "Authorized redirect URIs" is `[your-draw.io-hostname]/google`. For example, if you host draw.io at `https://drawio.example.com`, then "Authorized redirect URIs" whould be `https://drawio.example.com/google` and "Authorized JavaScript origins" would be `https://drawio.example.com`.

Set the values of the generated "Client ID" and "Client secret" into environment variables `DRAWIO_GOOGLE_CLIENT_ID` and `DRAWIO_GOOGLE_CLIENT_SECRET`. Also, set `DRAWIO_GOOGLE_APP_ID` environment variable to your Google App ID. APP ID is the number before the dash in the CLIENT ID. For example, if CLIENT ID is `123456789-abc...`, then APP ID is `123456789`

If you want to host a draw.io viewer also, you can create another client ID for the viewer. Viewer has read-only access to Drive files.

Set the values of the generated "Client ID" and "Client secret" into environment variables `DRAWIO_GOOGLE_VIEWER_CLIENT_ID` and `DRAWIO_GOOGLE_VIEWER_CLIENT_SECRET`. Also, set `DRAWIO_GOOGLE_VIEWER_APP_ID` environment variable to your Google App ID.

## Microsoft OneDrive

You will need to create and application in order to use MS Graph APIs. Follow the information on [how to register your app](https://docs.microsoft.com/en-us/graph/auth-register-app-v2) and [how to use the APIs](https://docs.microsoft.com/en-us/graph/use-the-api).

Once you registered your application, from Microsoft Azure UI, select your new app, then "Authentication". From Authentication, enter your redirect URIs. draw.io requires two redirect URIs `[your-draw.io-hostname]/microsoft` and `[your-draw.io-hostname]/onedrive3.html`. For example, if you host draw.io at `https://drawio.example.com`, then redirect URIs would be `https://drawio.example.com/microsoft` and `https://drawio.example.com/onedrive3.html`

In "Advanced settings" on the same page, enable "Access tokens" and "ID tokens" check boxes. To get the "Client secret", select "Certificates & secrets" page from the menu, then click "+ New client secret" button. Finally, from the "Overview" page in the menu, you can find the "Application (client) ID". 

Set the client ID and secret into environment variables `DRAWIO_MSGRAPH_CLIENT_ID` and `DRAWIO_MSGRAPH_CLIENT_SECRET`. If your Azure app is a single tenant, set `DRAWIO_MSGRAPH_TENANT_ID` to your tenant ID.
## Gitlab

Create a new OAuth app (Settings -> Applications). Set "Redirect URI" (e.g, `https://drawio.example.com/gitlab`) and "Scopes" (e.g, `api`, `read_repository
`, `write_repository`). Then, set the following environment variables, using information from the app, to enable Gitlab integration.

* `DRAWIO_GITLAB_ID`: Your Gitlab ID
* `DRAWIO_GITLAB_SECRET`: Your Gitlab Secret
* `DRAWIO_GITLAB_URL`: Your Gitlab base URL **without** any path, for example `https://gitlab.com` for SaaS or `https://gitlab.example.com` for self-hosted. The entrypoint appends `/oauth/token` itself when writing the server-side auth config, and the client uses this value as the base of the `/oauth/authorize` URL — adding a path here will produce broken URLs like `…/oauth/token/oauth/authorize`.

When `DRAWIO_GITLAB_URL` points at anything other than `https://gitlab.com` (i.e., any self-hosted GitLab), the entrypoint automatically appends `Editor.enableCustomGitLabUrl = true;` to `PostConfig.js`. Without that flag, `GitLabClient.authenticate()` short-circuits with an "access denied" error and shows a warning dialog before any OAuth request is made, so the OAuth flow looks broken even though the credentials are correct.

# AWS Deployment

You can deploy this docker compose easily to AWS ECS. Follow the instructions in this [tutorial](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-cli-tutorial-ec2.html) to install Amazonn ECS CLI, create a cluster, and deploy "self-contained" docker compose file to it. We recommend EC2 deployment.
You will need to change the port mapping to 80 and 443 in `docker-compose.yml` to support standard HTTP and HTTPS ports. Don't forget to allow access to these ports in the security group inbound rules. You must also set `DRAWIO_SERVER_URL` to your public deployment URL in order to have a fully functional deployment. Set the other environment variables as described above to enable other services and features as needed.

Refer to the main [README](https://github.com/jgraph/docker-drawio) file for how to configure **Let's Encrypt**.
