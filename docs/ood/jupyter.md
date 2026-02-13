
# Jupyter Lab
JupyterLab is a powerful, web-based interactive development environment for notebooks, code, and data. This instance runs in your browser for portability and ease of use.

This guide explains how to launch and use the **JupyterLab** app through Open OnDemand (OOD).

## Creating a python kernel
The site-wide python installation is quite bare-bones by design. In most cases, users will want to prepare a python virtual environment (venv) and kernel for jupyter before launching:

1. Load your desired python version
	```bash
	ml load python/{version}
	```
2. Create your venv in the desired location
	```bash
	python3 -m venv /path/to/venv
	```
3. Activate your venv
	```bash
	/path/to/venv/bin/activate
	```
4. Install desired packages
	```bash
	pip3 install numpy scipy ...
	# OR
	pip3 install -r requirements.txt
	```
5. Install necessary ipython packages
	```bash
	pip3 install ipykernel
	```
6. Add venv as a kernel in Jupyter
	```bash
    python -m ipykernel install --user --name=myenv --display-name "My Project Environment"
	```
7. (Optional) Deactivate venv
	```bash
	deactivate
	```

## Launching JupyterLab

1. Log in to muscadine’s [**Open OnDemand portal**](https://muscadine-node-1){target="_blank"}.
2. Navigate to **Interactive Apps → JupyterLab**.
3. Fill in the requested parameters:
    - **Cluster**: choose the compute resource you want.
    - **Number of cores / memory**: select the appropriate resources.
    - **Wall time**: specify how long you expect your session to run.
4. Click **Launch**.

Once your job starts, click **Connect to JupyterLab** to open the environment in a new browser tab.

## Using the environment

- JupyterLab provides an IDE-like interface for notebooks, terminals, and files.
- You can:
    - Create and edit **Python** or other supported notebooks.
	- In the top right, select your desired kernel (see [Creating a python kernel](#creating-a-python-kernel))
    - Open a new **Terminal** from the Launcher.
    - Manage files in your home or project directories.
    - Install additional Python packages in your environment (if permitted).

```{tip}
All notebooks and files are saved directly to your remote filesystem — no upload or download needed.
```

## Stopping your session

When finished:
1. Close the JupyterLab browser tab.
2. Return to the OOD dashboard.
3. Under **My Interactive Sessions**, click **Delete** or **Stop** to end the job and free up resources.

## Troubleshooting

- Jupyter launches but my custom kernel never launches
	- You skipped `module load python` which caused your venv to include the system python. recreate your venv after loading the python module.
- If JupyterLab fails to start, check:
    - The requested resources (wall time, cores) are available.
    - The **Interactive App** logs for errors (`View Output Log`).
- If your session disconnects, simply reconnect through **My Interactive Sessions**.

```{seealso}
For more help, contact your system administrators or consult your site’s OOD documentation.
```
