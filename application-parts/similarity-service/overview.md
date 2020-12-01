# Overview

## Similarity Service

This services is currently an experimental services. The main goal is to use NLP technologies to detect duplicate COLID entries and calculate how similar two COLID entries are. This service is written in `python 3.7` and provides a flask api. It is designed to train required NLP models itself. Please see under [setup](setup.md) for how to start this service. Under [architecture](architecture.md) an overview about the service implementation is given.

## Contibution

If a new dependency is needed, please add the specific version to the `requirements.txt` file.

## Code Style

This project use (in Pycharm integrated) the `PEP 8` code style standard.
