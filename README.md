# Implementation of NDSS'26 accepted paper "Bullseye: Detecting Prototype Pollution in NPM Packages with Proof of Concept Exploits"

## <a href="https://github.com/Madiba-Research/bullseye"> Click here for official repo </a>


Bullseye is a lightweight and fully-automated dynamic analysis tool, written in JavaScript, for detecting prototype pollution vulnerabilities in Node.js (NPM) packages. It is designed for large-scale analysis, capable of scanning over 50,000 packages in less than 8 hours. The tool leverages JavaScript reflection methods, the Node.js native virtual machine, and runtime observation oracles, in conjunction with a package’s own test suites and predefined payload seeds, to detect true positive vulnerabilities. Each confirmed finding is supported with a generated proof-of-concept exploit, ensuring accuracy and practical verification.

## Reference

This repository contains the implementation prototype of the approach described in our accepted paper at NDSS 2026. If you use this code, please cite our NDSS'26 paper.

## Directory Tree

```bash
.
├── benchmark  // The benchmark scripts and result artifacts
│   ├── bullseye.vs.ablation
│   ├── bullseye.vs.baselines
│   ├── Dataset
│   └── bullseye.vs.realworld // Real world analysis artifacts
├── bullseye.mjs // The actual analysis file run each time a package is invoked in the sandbox for the analysis
├── Dockerfile // use this file to build bullseye image
├── functionHandler.js   // utilities for handling functions enumeration
├── exploitGenerator.js  // generation the exploit candidates
├── testInputExtraction.js
├── fuzzPaterns.json  // exploit inputs patterns "the seed corpus"
├── gistPublisher.js // the github publisher file.
├── setup.js // setup the dataset benchmark folder
├── packageInit.js  // utility functions for handling package setup prior the analysis
├── run_refine.js  // refine the results prior publishing to github
└── run.js  // the orchestrator file
```

## For Reviewers
We published all our evaluation results in benchmark directory, seperated with directory for each research question:
- RQ 4: **`bullseye.vs.ablation`**
- RQ 1 & 3: **`bullseye.vs.baselines`**
- RQ 2: **`bullseye.vs.realworld`**

## Prerequisites

- [Node.js](https://nodejs.org/) (v20+ recommended)
- Docker (if using sandbox mode)
- npm dependencies installed (`npm install`)

## Usage

First, create the dataset file, containing the list of the packages in the format "package@version". npmPath used to specify that location to which the packages are installed

```bash
node setup.js --input "dataset/npm6588.json" --npmPath "/home/benchmark/npm6k"
```

Second, build the bullseye image (which act as the sandbox for each package analysis)

```bash
docker build -t bullseye:latest .
```

Finally, run the analysis tool, passing the dataset file to the script.

```bash
node run.js --input "dataset/npm6588.json" --npmPath "/home/benchmark/npm6k" --parallel 32 [options]
```

or analyze one package

```bash
node run.js --input "defaults-deep@0.2.3" --npmPath "/home/benchmark/npm6k" [options]
```

## Parameters

| Parameter                             | Description                                                                                                                                                                         | Default               | Example                                   |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- | ----------------------------------------- |
| `--input <file>`                      | Path to input file (JSON or TXT) with package list or single package string                                                                                                         | `dataset/npm47k.json` | `--input data.json`                       |
| `--format ["ss", "odgen", "default"]` | the format used for naming the package's folder, e.g., ss, odgen, leave it blank if you want the package to install normally in the current folder "if `npm init` is done already!" | `default`             | `--format "ss"`                           |
| `--tool <file>`                       | Path to the bullseye variation tool to run, used for run the variations of bullseye with disabled components                                                                        | `bullseye.mjs`        | `--tool baselines/fuzzproto_fnEnum.mjs`   |
| `--npmPath <file>`                    | Path to which the package is installed "excluding the npm generated path, i.e., node_modules/package@version"                                                                       | `${projDir}/npm`      | `--tool baselines/fuzzproto_fnEnum.mjs`   |
| `--install`                           | Install the package before running the tool (exist in setup.js)                                                                                                                     | `false`               | `setup.js --install "dataset/npm6k.json"` |
| `--vm`                                | Run entry points within Node.js VM                                                                                                                                                  | `true`                | `--vm=false`                              |
| `--sandbox`                           | Run the tool in a Docker container                                                                                                                                                  | `true`                | `--sandbox=false`                         |
| `--output <file>`                     | Export results to a file or stdout                                                                                                                                                  | `stdout`              | `--output file.json`                      |
| `--parallel <num>`                    | Number of concurrent packages to process                                                                                                                                            | `1`                   | `--parallel 8`                            |
| `--cveCheck`                          | Enable CVE check                                                                                                                                                                    | `false`               | `--cveCheck`                              |
| `--timeout <ms>`                      | Timeout for each package (milliseconds)                                                                                                                                             | `120000`              | `--timeout 300000`                        |
| `-h`, `--help`                        | Show help message                                                                                                                                                                   |                       | `--help`                                  |

## Input File Format

Bullseye supports dataset files with JSON format "as array of objects with at least `package_name` and `version` fields", and TXT format "One package per line, e.g., `lodash@4.17.21`".

## Output

- Results are printed to stdout or written to the specified output file "under raw-data subdirectory".
- Logs are saved in the `logs/` directory.

## Notes

- For large datasets, increase `--parallel` for faster processing.
- Bullseye analyze each package in a sanbox, use Dockerfile to build the image from which the containers will be created (requires Docker).
- The script supports both ESM and CommonJS npm packages.

## License

This project is licensed under the GNU GPL License.  
See the [LICENSE](./LICENSE) file for details.
