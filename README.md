# Kepler 16b

[![CI](https://github.com/opencaesar/kepler16b-example/actions/workflows/ci.yml/badge.svg)](https://github.com/opencaesar/kepler16b-example/actions/workflows/ci.yml)
[![Pages](https://img.shields.io/badge/Pages-HTML-blue)](http://opencaesar.github.io/kepler16b-example/) 

This is an example OML project for a hypothetical mission called Kepler 16b. For details, check this [tutorial](http://www.opencaesar.io/oml-tutorials/#tutorial2).

## Clone
```
  git clone https://github.com/opencaesar/kepler16b-example.git
  cd kepler16b-example
```

## Clean
```
./gradlew clean
```

## Build
```
./gradlew build
```

## Start Fuseki Server
```
./gradlew startFuseki
```

## Stop Fuseki Server
```
./gradlew stopFuseki
```

## Load to Fuseki Dataset
```
./gradlew owlLoad
```

Pre-req: A Fuseki server with a firesat dataset must be running at http://localhost:3030/firesat (see Start Fuseki above)  


## Run SPARQL Queries
```
./gradlew owlQuery
```

Pre-req: A Fuseki server with a firesat dataset must be running at http://localhost:3030/firesat (see Load to Fuseki Dataset)  


## Claude Code : 

1- Navigate to the Settings section in Claude Code.
2- Ensure that Code execution and file creation are enabled.
3- Go to Customize > Skills.
4- Toggle individual skills on or off as needed.
5- For organization-wide use, ensure that skills are enabled at the organization level in Organization settings > Skills.
For detailed instructions on creating and managing skills, refer to the Claude Help Center and Anthropic's documentation


## Copilot VS code : 
1- Click on Manage on the bottom left corner
2- Click on Settings in manage menu
3- Type "Agent skills" in the search bar
4- check mark "Chat : Use Agent Skills"

## Codex : 

1- Install Codex CLI: Ensure you have the Codex CLI installed on your system.
2- Create a Skills Directory: Create a directory named skills within your Codex home directory.
3- Create a SKILL.md File: Inside the skills directory, create a file named SKILL.md and add the necessary metadata and instructions for your skill.
4- Install the Skill: Use the built-in skill installer in Codex CLI to install your skill from a repository or manually.
5- Enable the Skill: Mention the skill name in your Codex prompt to enable it for use.

For detailed instructions and examples, refer to the 2026 Install and Usage Guide provided by OpenAI. This guide covers everything from installing Codex CLI to authoring and deploying skills across various platforms



