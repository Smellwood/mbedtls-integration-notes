# mbedtls-integration-notes

Shared working notes between [Smellwood](https://github.com/Smellwood) and [doctor-elwood-phd](https://github.com/doctor-elwood-phd) for the Mbed TLS open-PR integration and conflict report.

This repository is **not affiliated with** the [Mbed-TLS](https://github.com/Mbed-TLS) project or its maintainers.

## Purpose

This is a coordination space and a place to drop raw results: merge/conflict maps, build outcomes, test logs, and reproduction notes. The public report itself will go upstream later as a single issue, after both of us have reviewed it. Individual fixes still go upstream one pull request at a time, with original authors credited.

## Layout

Raw results live as Markdown under:

- [`results/development/`](results/development/) — development branch
- [`results/mbedtls-3.6/`](results/mbedtls-3.6/) — `mbedtls-3.6` (LTS)
- [`results/mbedtls-4.1/`](results/mbedtls-4.1/) — `mbedtls-4.1` (LTS)

## How to talk

- Open one GitHub issue per topic and keep the discussion in that thread.
- Add or update results with a pull request (preferred) or a commit on the default branch if you have write access.
- Tag the other contributor when a result is ready for independent reproduction.

AI-assisted, human-directed.
