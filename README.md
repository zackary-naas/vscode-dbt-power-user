# vscode-dbt-power-user

![Last updated](https://img.shields.io/visual-studio-marketplace/last-updated/innoverio.vscode-dbt-power-user) ![Version](https://img.shields.io/visual-studio-marketplace/v/innoverio.vscode-dbt-power-user) ![Installs](https://img.shields.io/visual-studio-marketplace/i/innoverio.vscode-dbt-power-user) ![Build passing](https://github.com/innoverio/vscode-dbt-power-user/workflows/.github/workflows/ci.yml/badge.svg)

This extension makes vscode seamlessly work with [dbt](https://www.getdbt.com/).

If you need help with setting up or any other functionality, message us in the dbt community slack channel [#tools-dbt-power-user](https://getdbt.slack.com/archives/C05KPDGRMDW).

Main features:

- [Generate models from your source definitions](#sourcegen)
- [Query result set visualization](#sqlrunner)
- [Lineage of dbt models](#lineage)
- [Execute all or individual model tests](#modeltester)
- [Go to the definition of any models, macros, sources and docs.](#gotodef)
- [Edit documentation of your model and its columns in a dedicated panel](#doceditor)
- [View column details, go to documentation editor and run dbt docs generate.](#doctreeview)
- [Autocompletion of models, macros, sources and docs](#autocomplete)
- Run a model through the play button of the document
- Build a model through the paper plane button (next to the play button)
- dbt update notifications
- [dbt logs viewer (force tailing)](#logtailer)
- [Run workspace scan and see the issues across all projects in the workspace](#projectscan)

This extension is using the Python extension to detect Python interpreters that are installed in standard locations. See [Python Environments](https://code.visualstudio.com/docs/languages/python#_environments).

This extension is fully compatible with the remote extension. See [Visual Studio Code Remote - Containers](https://code.visualstudio.com/docs/remote/containers) and [Visual Studio Code Remote - WSL](https://code.visualstudio.com/docs/remote/wsl).

Please let us know about any bugs or feature requests through the github issues.

If you require additional support, feel free to reach out on the dbt Community Slack Channel [#tools-dbt-power-user](https://getdbt.slack.com/archives/C05KPDGRMDW)

If you want to contribute to the extension, let us know and we will help set you up.

## How to setup the extension

### Associate your .sql files the jinja-sql language

You should associate your .sql files with the jinja-sql language by configuring in Preferences > Settings

![Associations](./media/images/associations.png)

or add the following in settings.json:

```
    "files.associations": {
        "*.sql": "jinja-sql"
    },
```

### Select the Python interpreter that has dbt installed

Ensure that the Python interpreter selection is always visible for ease of use:

![Enable python interpreter selection](./media/images/enable-python-interpreter-visibility.gif)

Make sure that you select the interpreter that has dbt installed. In my case it is virtual environment in the project itself.

![Select the right python interpreter](./media/images/select-python-interpreter.gif)

Avoid using the setting `dbt.dbtPythonPathOverride` unless using Meltano, the extension depends on the Python interpreter for visual code compatible environment variable parsing.

When you set the Python interpreter, the extension will try to detect dbt and you should be able to make use of the features listed below.

### Select the custom dbtRunner.

In case you want to use a custom runner. You can overwrite the import string for it.

For an example, you have module `my_custom_runner`:

```python
from dbt.cli.main import dbtRunner

class dbtCustomRunner(dbtRunner):
    def invoke(self, *args, **kwargs):
        print("Hello from overwritten runner!")
        print("Staring invoke...")
        return dbtRunner.invoke(self, *args, **kwargs)
        print("Invoke finished!")
```

`dbt.dbtCustomRunnerImport` = `from my_custom_runner import dbtCustomRunner as dbtRunner`

### Environment variables

This extension supports environment variables in various ways;

#### Environment variables setup outside of Visual Code (.zshrc, .bashrc, ...)

These environment variables will be passed to all operations of the extension. Note that the environment variable should be valid for all your dbt projects. For example `DBT_PROFILES_DIR` can be set to `.`, that way dbt will lookup the profiles.yaml file inside the root of the dbt project.

#### Environment variables through `python.envFile`

The extension also loads an environment variable definitions file identified by the python.envFile setting. The default value of this setting is `${workspaceFolder}/.env`.

This way supports all Visual Code variable substitution patterns and is best practise. Read all about [environment variables](https://code.visualstudio.com/docs/python/environments#_environment-variables) supported by the Visual Code Python extension

#### Environment variables setup for the terminal

The extension will read any vscode configurations in `.vscode/settings.json` and pass them to all operations of the extension:

```
"terminal.integrated.env.[osx|windows|linux]": {
    "DBT_PROFILES_DIR": "."
}
```

Note that Visual Code variable substitution is not supported except the environment variable pattern `${env:*}` and `${workspaceFolder}`.

### Query results visualization

Your database may not support standard SQL LIMIT statements like `SELECT * from table LIMIT 10`.

You can override this default behaviour through `dbt.queryTemplate`.

Please make a PR if you find that you need to change `dbt.queryTemplate` for your favourite adapter and help the community.

#### `dbt.queryTemplate` for Oracle

Change to `select * from ({query})\n where ROWNUM <= {limit}`

#### `dbt.queryTemplate` for MS SQL, Synapse

Change to `set rowcount {limit}; {query}` or `{query}\n order by 1 OFFSET 0 ROWS FETCH FIRST {limit} ROWS ONLY` (note that your query can't have an order by clause).

### Format your dbt SQL with `sqlfmt`

#### Install `sqlfmt`

Install sqlfmt through running `pip install 'shandy-sqlfmt[jinjafmt]'`. Find more about sqlfmt in their [docs](https://sqlfmt.com/).

#### Configure `dbt.sqlFmtPath` or `dbt.sqlFmtAdditionalParams`

You can configure the path to sqlfmt through `dbt.sqlFmtPath` and you can configure additional parameters through `dbt.sqlFmtAdditionalParams`.

#### Usage

Please select "dbt Power User" (extension id:`innoverio.vscode-dbt-power-user`) as the default formatter. You can do this either by using the context menu (right click on a open dbt model in the editor) and select "Format Document With...", or you can add the following to your settings:

```json
  "[jinja-sql]": {
    "editor.defaultFormatter": "innoverio.vscode-dbt-power-user"
  }
```

#### Format on save

You can enable format on save for python by having the following values in your settings:

```json
  "[jinja-sql]": {
    "editor.defaultFormatter": "innoverio.vscode-dbt-power-user",
    "editor.formatOnSave": true
  }
```

### Configure `dbt.allowListFolders` (Optional)

To specify project folders explicitly and control which projects are included in the build process, you can configure the `dbt.allowListFolders` setting. This can be particularly useful when you have a large number of projects in the same workspace. You can define workspace-relative paths to include as follows:

```json
"dbt.allowListFolders": [
  "folder1",
  "folder2"
]
```

If this setting is not specified or is left empty, no filtering will be applied, and all projects will be parsed.

### Telemetry

Telemetry is used for error and usage reporting in order to make the extension better.

[Find out how to disable telemetry in visual code docs](https://code.visualstudio.com/docs/getstarted/telemetry#_disable-telemetry-reporting).

## Features at work

### <a id="lineage">Lineage of current model</a>

![See the parents and children of your model](./media/images/lineage.png)

### <a id="sourcegen">Generate a model from your source definition</a>

![Generate a model from your source definition](./media/images/generate-model-from-source.gif)

You can select a file name template and prefix in the settings.

### <a id="doceditor">Edit the documentation from your model</a>

![Edit the documentation from your model](./media/images/documentation-editor.png)

### <a id="doctreeview">View Column details of current model</a>

![Tree view of of columns in model documentation](./media/images/demo-doc-treeview.gif)

Clicking the table name will route you to the YML schema file if it exists:

![Tree view of of columns in model documentation](./media/images/demo-doc-view-clickable.gif)

You can run Generate Documentation Schema or Trigger `dbt docs generate` from the documentation tab.
This feature relies on a defined schema in a YML file. It extracts data from both the catalog.json, and the manifest.json.

### <a id="sqlrunner">Visualize the result set of your model (Use CMD+ENTER (mac) or CTRL+ENTER (win))</a>

![Use ctrl+enter or cmd+enter to retrieve the result set of your model](./media/images/visualize-result-set.gif)

### <a id="compiler">See the compiled query of your model (Use CMD+' (mac) or CTRL+' (win))</a>

![Use ctrl+' or cmd+' to see the compiled sql of your model](./media/images/compile.gif)

### <a id="modeltester">Execute model tests</a>

![Test your model](./media/images/tests.gif)

### <a id="modelgraph">See the model graph</a>

![See the graph and execute parent or children models](./media/images/graph.gif)

### <a id="gotodef">Go to definition</a>

![Go to model definition](./media/images/definition-model.gif)

![Go to macro definition](./media/images/definition-macro.gif)

![Go to source definition](./media/images/definition-source.gif)

![Go to doc definition](./media/images/definition-doc.gif)

### <a id="autocomplete">Autocompletion</a>

![Autocomplete model](./media/images/autocomplete-model.gif)

![Autocomplete macro](./media/images/autocomplete-macro.gif)

![Autocomplete source](./media/images/autocomplete-source.gif)

![Autocomplete source](./media/images/autocomplete-doc.gif)

### <a id="logtailer">dbt logs force tailing</a>

![dbt logs](./media/images/dbt-log.gif)

### <a id="projectscan">Project Health Check</a>

#### Prevent issues from getting shipped to production!

Our new feature lets you swiftly scan all dbt projects in your workspace, bringing any issues straight to the problems panel. Depending on your team's conventions, we've classified potential discrepancies as warnings or just informational pointers.

#### What the Scanner Checks For:

- **Missing schema.yml Files**:

  - If some models aren’t documented, they'll show up here.
  - _Note_: We give seeds or ephemeral models a pass, as they're often traditionally undocumented.

- **Undocumented Columns**:

  - For models with a schema.yml but with some columns left undocumented.
  - This typically points to a mismatch between the actual database model and its documentation. It's more of a heads-up than an error.

- **Extra Columns**:

  - Over time, columns might get dropped from models but linger in the schema.yml.
  - Our scanner catches this. Or, maybe it's just a typo in the schema.yml

- **Models Absent in the Database**:
  - We'll flag seeds and models missing from the database.
  - Maybe it's an unused model ripe for removal, or perhaps it’s a fresh model that's yet to make its debut in the database.

![projectscan](./media/images/project-scan.gif)
