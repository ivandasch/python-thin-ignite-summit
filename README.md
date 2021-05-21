# python-thin-ignite-summit
Python thin client ignite summit talk.

# Slides
See slides [here](https://ivandasch.github.io/python-thin-ignite-summit/)

If you want to compile slides by yourself, do following (`docker` must be installed.)
```bash
cd slides
make slides
```

# Running examples
1. Start ignite in docker:
```bash
docker-compose up -d
```
2. Install [poetry](https://github.com/python-poetry/poetry#installation)

3. Install dependencies
```bash
poetry install
```
4. Run jupyter
```bash
poetry run jupyter notebook
```
Notebook with examples are located in `./examples` subfolder.
