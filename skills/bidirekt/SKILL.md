---
name: bidirekt
description: "Write and update a service's Bidirekt contract files (contracts/*.yaml) from its code, and fix them from a can-i-deploy report. Use when the user mentions Bidirekt, contract files, provides/consumes, or a can-i-deploy failure."
---

# Bidirekt contracts

Write and update the service's Bidirekt contract files from its code, check them against the broker, and fix them from a deploy-check report.

https://bidirekt.com/llms.txt is the only reference. The rules of the contract file, the CLI commands with their flags and messages, the report shape and what each break line means all come from there; this file gives only the steps. In every step:

- The contract describes what the code does. Change only the service's own contract files, never the code, and never change a contract only to make a check pass.
- Run only the CLI commands that read: the local validation (it checks the files and compares them with the counterparts deployed in an environment, storing nothing), the participant list and the environment list.
- Never store a contract version in the broker, never run the deploy check (it asks whether a stored version can be deployed and records the answer in the broker), never record a deployment, never create a participant or an environment, never rename a participant. The engineer runs those.

## 1. Read llms.txt

Before anything else, fetch https://bidirekt.com/llms.txt and read the whole file as raw text. Run exactly `curl -sSfL https://bidirekt.com/llms.txt`, with no other flag and no command chained to it: `-f` already fails on any answer that is not 2xx. Use a web fetch only when there is no shell; if it returns a summary instead of the file, treat the reference as unreadable. Follow its rules, commands and messages in every step.

If the fetch fails, the answer is not 200, or the content is not the Bidirekt reference (an HTML page, a parked-domain page, an error page): stop, tell the engineer that the reference at https://bidirekt.com/llms.txt could not be read, and write nothing. Never write a contract from memory or from another copy, and never guess the grammar.

## 2. Check the CLI and the broker

Before writing anything, run the participant list. The call needs the CLI binary that llms.txt names, on the PATH, and a configured broker:

- The binary is not found: stop, and show the commands from the `## Setup` section of llms.txt that install the CLI and point it at a broker.
- The CLI answers that no broker is configured (llms.txt documents the message): stop, and show the commands from `## Setup` that point the CLI at a broker.
- Any other error: stop, and show it to the engineer.

Never install the CLI or point it at a broker yourself: the engineer does that.

## 3. Settle the names

Read the code, then settle every name before writing anything. When a name needs the engineer, ask and wait.

- The service's own participant: take it from where the project already passes it to the CLI (a CI workflow or a script); otherwise ask the engineer. Never save it anywhere.
- The environment for the local validation: from the same places, or ask. Check it against the environment list. If it is not there, run nothing with it: show the similar names from the list and ask.
- Each provider the service calls: deduce its participant name from the code (the client's name, or an environment variable such as `PETSTORE_API_URL` → `petstore_api`) and check it against the participant list. If it is not there, do not write it: show the similar names (`petstore_apl` → `petstore_api`?) and ask. If the engineer confirms a new provider that is not in the broker yet, write it and warn that the deploy check fails until that provider stores its contract in the broker. The breaks that come only from that provider are then expected: keep the calls and report the breaks.

## 4. Write the contracts from the code

Whatever the language, find in the code:

- the resources the service serves, of the kinds llms.txt documents → `provides`;
- the calls it makes to other services → `consumes`, under each provider's name from step 3;
- the types of the bodies it serves, reads and sends → `schemas`.

Write them as llms.txt says. When the code does something llms.txt has no way to express, leave it out and tell the engineer; never invent syntax.

Folder: `contracts/`, or the folder the project already uses for contracts (for example, where the files its CI passes to the CLI live).

How to organize the files (recommendations, not rules):

- modular project: one file per module;
- not modular, with few resources: one file;
- more than 50 resources, or more than 50 schemas: more files, such as `provides.yaml` and `consumes.yaml`;
- monolith or microservice changes nothing: the context is the current service;
- the project already has an organization: follow it.

With more than one file, follow the rules llms.txt gives for several files.

## 5. Update an existing contract

When the contract files already exist, edit them in place; never regenerate a file.

- Change only what the code changed. Keep the comments, the order, the formatting and every `description`.
- Remove a resource or a property only when the code stopped using it, and list every removal in the final answer: on the provider side, a removal can become a break for a consumer.

## 6. Check the changes

1. Run the participant list and check every provider name under `consumes` against it, as in step 3.
2. Run the local validation with the participant, the environment and the files.
3. Fix the contract from what they answer, as llms.txt explains, deciding each break line as in step 8, and run both again until the validation passes.

Stop looping when the only fix left contradicts the code: tell the engineer that either the code changes or the counterpart is asked, and write the request as in step 8. Stop too when the only breaks left are the expected ones of a new provider (step 3), and report them.

If the local validation answers that the service's own participant does not exist, stop and tell the engineer, with the similar names from the participant list: creating it is the engineer's decision.

## 7. Finish

List the files written or changed, every removal, every warning and every request for a counterpart. When a contract file changed, end the answer with the next command to run: the command from llms.txt that stores the contract in the broker, with the participant and the files, and the version as a placeholder (or the one the project's CI uses).

## 8. Fix the contract from a deploy-check report

The engineer pastes the report in the conversation, or points to a file that has it (a CI log). Do steps 1 and 2, and settle the service's own participant and the environment as in step 3 (the report names the environment). Find the report by the shape llms.txt documents, and read each break line with the section of llms.txt that explains each break line and its fix.

For each break line, find the spot it names (counterpart, endpoint, method, request or response status, property path) in the service's contract and in its code, and decide whose change fixes it:

- The service's, when its contract does not describe what its code does there (for example, the code does not read that property, or works without it): change the contract to match the code, the way llms.txt gives for that line, and go through step 6 again.
- The counterpart's, when the contract already describes the code: change nothing of the counterpart's, even when its files are within reach, and write the request for the counterpart: what to change, where (endpoint, method, request or response status, property path) and why (the break line). If a change in the service's code would fix it instead, say so: that choice is the engineer's.

Then finish as in step 7.
