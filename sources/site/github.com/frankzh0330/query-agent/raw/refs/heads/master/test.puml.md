# Source: https://github.com/frankzh0330/query-agent/raw/refs/heads/master/test.puml

@startuml 'https://plantuml.com/sequence-diagram autonumber == Initialization == Alice -> Bob: Authentication Request Bob --> Alice: Authentication Response == Repetition == Alice -> Bob: Another authentication Request Alice <-- Bob: another authentication Response @enduml