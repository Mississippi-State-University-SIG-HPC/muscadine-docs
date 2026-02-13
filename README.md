# Muscadine Documentation Source Repository

```
$$\      $$\                     Welcome to:         $$\$$\
$$$\    $$$ |                                        $$ \__|
$$$$\  $$$$ $$\   $$\ $$$$$$$\ $$$$$$$\$$$$$$\  $$$$$$$ $$\$$$$$$$\  $$$$$$\
$$\$$\$$ $$ $$ |  $$ $$  _____$$  _____\____$$\$$  __$$ $$ $$  __$$\$$  __$$\
$$ \$$$  $$ $$ |  $$ \$$$$$$\ $$ /     $$$$$$$ $$ /  $$ $$ $$ |  $$ $$$$$$$$ |
$$ |\$  /$$ $$ |  $$ |\____$$\$$ |    $$  __$$ $$ |  $$ $$ $$ |  $$ $$   ____|
$$ | \_/ $$ \$$$$$$  $$$$$$$  \$$$$$$$\$$$$$$$ \$$$$$$$ $$ $$ |  $$ \$$$$$$$\
\__|     \__|\______/\_______/ \_______\_______|\_______\__\__|  \__|\_______|
```

## Building

This repo is designed to be built with Shibuya Sphinx theme. Steps are as follows

```bash
# Recommended: creat a venv
python -m venv /var/tmp/sphinx
. /var/tmp/sphinx/bin/activate

# install requirements
pip install -r requirements.txt

# build docs
make html
```
