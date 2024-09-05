# Standardizing Model-Related Flags in CLI and Libraries

## Introduction
- **Purpose**: Define the standard practices for model-related flags used in CLI tools and libraries to ensure consistency, usability, and maintainability.
- **Scope**: Covers all model-related flags used in the command-line interface (CLI) and libraries within the project.
- **Audience**: Developers, maintainers, and contributors involved in the development and usage of the CLI tools and libraries.

## Summary
This proposal aims to standardize the use of model flags to ensure consistent behavior across all commands and subcommands.

## Motivation
Currently there are 10+ instances across all major commands and subcommands that accept either `--model` , `--model-path`, 
`--model-name`, or `--model-dir` all of which serve slightly different purposes, and might handle different use cases including local vs remote models, relative vs absolute paths etc. This issue seeks to propose a solution that will standardize the use of model flags and make them behave consistently across all commands and sub-commands.

## Problem Statement
This aims to streamline the number of model related flags available, while establishing consistency and predictability between their uses and handling 3 separate use-cases: absolute paths, relative paths and remote repository names. 

## Proposal
- Retain `--model` and deprecate all other model related flags
- `--model` should accept both paths, as well as str for repo names
- First `--model` should assume whatever is passed into it is an absolute path, and should check against root to determine if that path exists
    - If it exists, we should run an additional check to determine if it points to a safetensor or gguf model (use existing `is_model_safetensors` and `is_model_gguf` checks for this)
    - If does not exist, or exists but is not determined to be a valid model then move on
- If not abspath, assume the path supplied is a relative path. Expand it by appending it `~/.local/cache/instructlab/models` and check if the path exists.If exists and model is found, use it. If not, move on
- If supplied content is neither an abspath, nor a relative path - assume it is the name of a remote repo on HF and download it
    - alternatively, we can error out here and require that user download the model explicitly via ilab model download

This would standardize the behavior of the `--model` flag across all the commands that it appears in. There could be a dedicated model resolver function that implements the above described process.

The only exception may be ilab model download which contains a `--model-dir` flag which could stand to benefit from being renamed to `--destination`.

In addition to standardizing the flag itself, we must also standardize what gets passed INTO the flag, i.e the value passed to
`--model`
That is where PR: #1895 comes in. This PR aims to standardize the format in which users reference their models. This PR will make it so users always reference their models via the complete URL of wherever the model is hosted (e.g quay.io/ai-lab/models/granite-7b-lab) and that models are always downloaded into `~/.local/cache/instructlab/models` under sub-directories that follow the same structure as their URL ~/.local/cache/instructlab/models/quay.io/ai-lab/models/granite-7b-lab/v1.1, for e.g. This will ensure models are always referenced in a consistent way across the entire workflow (which will double up as relative paths for where those models are stored)

Proposed combined approach will resovle all associated issues in #1871. 

## How would backwards-compatability be handled?
All other model flags will be deprecated for a couple releases and called out in the release notes. They will eventually be removed.
The fields in the config file will need to be updated to match `--model`, which might be a breaking change and may warrant bumping the config version. This might require implementation of some kind of automatic config conversion mechanism between versions