# Checkmate Examples
This repository includes examples for working with Checkmate from Red Pill Analytics. Checkmate enables Continuous Delivery for products or platforms that don't naturally support it.

### Layout
The repository is designed to have a directory for each Checkmate plugin located in the project root (called the *plugin directory*), each containing one or more example projects located in separate directories (called the *project directory*.) The project directory contains a **README.md** file that contains the Quickstart description and exercises for working with Checkmate.

Currently, we only have a single Quickstart configured: [Checkmate for OBI using OBIEE 12.2.1.2](obi/sample-12c/README.md).

# Checkmate Studio
![studio17](studio17.png)
### Installation
To get started with Checkmate Studio simply download and install the latest version from the Red Pill Analytics downloads page: [http://redpillanalytics.com/checkmate-getstarted/](http://redpillanalytics.com/checkmate-getstarted/) for your chosen operating system.
### Getting Started
You will see a welcome screen and you will want to choose 'New project' to create a new OBI checkmate project. You will need to fill out the appropriate settings before you can begin using the application: source base, domain/middleware home, OBIEE version, source base type, and checkmate version.
### Executing Tasks
To execute a task such as metadata import or catalog export, simply click anywhere or click on the plus icon in the shortcut area located along right side of the project window. Then choose your desired task, fill in relevant data and click 'execute task'. Output will be displayed below the card. A green corner represents a success and a red corner means an error occurred.

*Note: If your task requires either Git credentials or OBIEE credentials a small modal dialog will present itself where you can enter the appropriate credentials. If you entered the wrong credentials you can change these at anytime using the shortcut area along the right side of the project window.*
### Using Git Integration
You can use the Git pane at the bottom to commit or discard code changes to the current source control branch (which can be seen in brackets in the window title).
### Further Help/Support
You can use any of the support or help links in the application to seek further assistance or help in using the Checkmate Studio application.