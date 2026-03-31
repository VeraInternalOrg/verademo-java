# VeraDemo - Blab-a-Gag

### Notice

This project is intentionally vulnerable! It contains known vulnerabilities and security errors in its code and is meant as an example project for software security scanning tools such as Veracode. Please do not report vulnerabilities in this project; the odds are they’re there on purpose :) .

## About

Blab-a-Gag is a fairly simple forum type application which allows:

- Users can post a one-liner joke.
- Users can follow the jokes of other users or not (listen or ignore).
- Users can comment on other users messages (heckle).

### URLs

- `/feed` shows the jokes/heckles that are relevant to the current user.
- `/blabbers` shows a list of all other users and allows the current user to listen or ignore.
- `/profile` allows the current user to modify their profile.
- `/login` allows you to log in to your account
- `/register` allows you to create a new user account
- `/tools` shows a tools page that shows a fortune or lets you ping a host.

## Run

If you don't already have Docker this is a prerequisite.

    docker run -p 8080:8080 veracode/vulnerable-verademo-java

Navigate to: `http://127.0.0.1:8080`.

## Exploitation Demos

See the [DEMO_NOTES](DEMO_NOTES.md) file for information on using this application with the various Veracode scan types.

Also see the `docs` folder for in-depth explanations of the various exploits exposed in this application.

## CI System Demos

There are build files for various CI systems included as part of this application. Often there are several sample build files for each CI system, but there will always be at least an 'essentials' file that shows the basic steps to get the application packaged and scanned with Veracode's technology.

Note that there are some secrets that need to get set in the build files. These might vary a bit between CI systems, but generally:

- `VERACODE_API_ID` & `VERACODE_API_KEY`: The API credentials of the Veracode user account used to run the scan. See [here](https://docs.veracode.com/r/c_api_credentials3).
- `SRCCLR_API_TOKEN`: The token needed for the agent-based SCA scanner. See [here](https://docs.veracode.com/r/Integrate_Veracode_SCA_Agent_Based_Scanning_with_Your_CI_Projects).

| CI System     | "Essentials" File                                                  |
| ------------- | ------------------------------------------------------------------ |
| GitHub        | `.github/workflows/the-essentials.yml`                             |
| Azure Dev Ops | `azure-pipelines.yml`                                              |
| Jenkins       | `Jenkinsfile`                                                      |
| GitLab        | `.gitlab-ci.yml`                                                   |
| AWS           | `AWS-CodeStar.md` (directions for configuring AWS CodeStar builds) |
| BitBucket     | `bitbucket-pipelines.yml`                                          |

## Technologies Used

- Spring boot
- MariaDB

## Development

To build the original single-container image run this:

    docker pull mariadb:10.6.2
    docker build --no-cache -t verademo .

To run the original single-container image locally:

    docker run --rm -p 8080:8080 --name verademo verademo

Then register as a new user and add some feeds

### Docker Compose development

There is now a compose-based local workflow that runs the database and application in separate containers.

Start the stack:

    docker compose up --build

Then, in a second terminal, start file watching:

    docker compose watch

What this does:

- Changes under `app/src/main/java`, `app/src/main/resources`, and `app/src/main/webapp` are synced into the app container and the app container is restarted.
- Changes to `app/pom.xml` or `maven-settings.xml` trigger an image rebuild.
- The application is available at `http://127.0.0.1:8080`.
- The database is available on `127.0.0.1:3306` for local tools or for running the app directly on the host.

This is not true JVM hot reload. `docker compose watch` syncs files and restarts the app container so `mvn spring-boot:run` recompiles on startup.

The compose workflow also upgrades the legacy SQL seed users to BCrypt on app startup. That keeps login behavior aligned with the application code even when the MariaDB container initializes from the SQL dump.

### Fastest local edit loop

For the fastest feedback loop, run only the database in Docker and run the Spring app on your host machine.

Start the database container:

    docker compose up -d db

Run the app from the `app` directory:

    DB_HOST=127.0.0.1 DB_PORT=3306 DB_NAME=blab DB_USER=blab DB_PASSWORD='z2^E6J4$;u;d' mvn spring-boot:run

This uses Spring Boot DevTools, which is already in the Maven build, so code and resource changes are usually picked up faster than a full container restart.

### Using nodemon

Yes, you can use `nodemon` as a generic watcher for a Java project, but it will restart the Maven process rather than hot-swap compiled classes.

Example from the `app` directory:

    npx nodemon --watch src/main --ext java,jsp,properties,xml --exec "mvn spring-boot:run"

That works, but it is usually slower and less efficient than running `mvn spring-boot:run` directly with Spring Boot DevTools. For this project, `docker compose watch` or host-side `mvn spring-boot:run` are the better default choices.
