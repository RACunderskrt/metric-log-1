# Exercice 5

main.yml
```yml
name: Java Report for BankApplication

on:
  push:
    paths:
      - 'BankApplication/**'
  pull_request:
    paths:
      - 'BankApplication/**'
  workflow_dispatch: 

jobs:
  java-report:
    runs-on: debian-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Generate Java file report
      run: |
        # dossier à analyser
        DIR="BankApplication"

        # vérifier qu'il y a au moins un .java
        JAVA_FILES=$(find "$DIR" -type f -name "*.java")
        if [ -z "$JAVA_FILES" ]; then
          echo "No .java files found in $DIR"
          exit 1
        fi

        # vérifier qu'il n'y a pas de .class
        CLASS_FILES=$(find "$DIR" -type f -name "*.class")
        if [ ! -z "$CLASS_FILES" ]; then
          echo "Compiled .class files found in repo (bad practice):"
          echo "$CLASS_FILES"
          exit 1
        fi

        # nombre total de fichiers .java
        NUM_FILES=$(echo "$JAVA_FILES" | wc -l)

        # total lignes de code
        TOTAL_LINES=$(xargs wc -l <<< "$JAVA_FILES" | tail -1 | awk '{print $1}')

        # créer le rapport
        REPORT="java-file-report.txt"
        echo "Number of .java files: $NUM_FILES" > "$REPORT"
        echo "Total lines of code: $TOTAL_LINES" >> "$REPORT"
        echo "" >> "$REPORT"
        echo "Per-file breakdown:" >> "$REPORT"

        while read -r FILE; do
          LINES=$(wc -l < "$FILE")
          echo "$FILE – $LINES" >> "$REPORT"
        done <<< "$JAVA_FILES"

        echo "Java file report generated:"
        cat "$REPORT"

    - name: Upload report
      uses: actions/upload-artifact@v3
      with:
        name: java-file-report
        path: java-file-report.txt
```