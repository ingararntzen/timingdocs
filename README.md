# timingdocs

Documentation for [timingsrc (v3)](https://webtiming.github.io/timingsrc/),
hosted on [https://timingsrc.readthedocs.io](https://timingsrc.readthedocs.io).


# virtualenv

A Python virtual environment with sphinx and other dependencies
needed for project startup and local builds.


Create Python environment

```sh
python3 -m venv venv
```

Activate Python environment

```sh
source venv/bin/activate
```

Install sphinx

```sh
pip install --upgrade pip
pip install -r requirements.txt
```


Deactivate Python environment

```sh
deactivate
```

# build

```sh
cd docs
make html
```

# view

```sh
google-chrome docs/build/html/index.html
```
