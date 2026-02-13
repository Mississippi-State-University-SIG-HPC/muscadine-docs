# VSCode
VSCode is a lightweight, open-source IDE developed by Microsoft. This instance runs in your browser for portability.

This guide explains how to launch and use the **VS Code (code-server)** app through Open OnDemand (OOD).

## Launching code-server

1. Log in to muscadine’s [**Open OnDemand portal**](https://muscadine-node-1){target="_blank"}.
2. Navigate to **Interactive Apps → VS Code**
3. Fill in the requested parameters:
    - **Cluster**: choose the compute resource you want.
    - **Number of cores / memory**: select the appropriate resources.
    - **Wall time**: specify how long you expect your session to run.
4. Click **Launch**.

Once your job starts, click **Connect to VS Code** to open the editor in a new browser tab.

## Using the editor

- The environment is the same as desktop VS Code, but runs remotely on the compute node.
- You can:
    - Open and edit files in your home or project directories.
	- Change the working directory with {kbd}`Ctrl`+{kbd}`K`, {kbd}`Ctrl`+{kbd}`O`
    - Use the **Terminal** ( {kbd}`Ctrl`+ {kbd}`Shift`+ {kbd}`C`) for command-line access.
    - Install VS Code extensions

```{tip}
All file edits are saved directly to your remote filesystem — no upload or download needed.
```

## Stopping your session

When finished:
1. Close the VS Code browser tab.
2. Return to the OOD dashboard.
3. Under **My Interactive Sessions**, click **Delete** or **Stop** to end the job and free up resources.

## Troubleshooting

- If VS Code fails to start, check:
    - The requested resources (wall time, cores) are available.
    - The **Interactive App** logs for errors (`View Output Log`).
- If your session disconnects, simply reconnect through **My Interactive Sessions**.

```{seealso}
For more help, contact your system administrators or consult your site’s OOD documentation.
```
