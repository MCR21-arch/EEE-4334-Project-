#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_INPUTS 16
#define MAX_OUTPUTS 16
#define MAX_ROWS 1024

typedef struct {
    int num_inputs;
    int num_outputs;
    char inputs[MAX_INPUTS][10];
    char outputs[MAX_OUTPUTS][10];
    char table[MAX_ROWS][MAX_INPUTS + MAX_OUTPUTS + 2]; // Input + Output + null terminator
    int row_count;
} PLA;

// Parse the PLA file
void parse_pla(const char *file_name, PLA *pla) {
    FILE *file = fopen(file_name, "r");
    if (!file) {
        perror("Error opening file");
        exit(EXIT_FAILURE);
    }

    char line[256];
    while (fgets(line, sizeof(line), file)) {
        // Ignore empty lines and comments (lines starting with ".")
        if (line[0] == '.' || line[0] == '\n') {
            continue;
        }

        if (strncmp(line, ".i", 2) == 0) {
            sscanf(line, ".i %d", &pla->num_inputs);
        } else if (strncmp(line, ".o", 2) == 0) {
            sscanf(line, ".o %d", &pla->num_outputs);
        } else if (strncmp(line, ".ilb", 4) == 0) {
            // Parse input labels
            char *label_start = line + 5; // Skip ".ilb "
            for (int i = 0; i < pla->num_inputs; i++) {
                sscanf(label_start, "%s", pla->inputs[i]);
                label_start += strlen(pla->inputs[i]) + 1; // Move to the next label
            }
        } else if (strncmp(line, ".ob", 3) == 0) {
            // Parse output labels
            char *label_start = line + 4; // Skip ".ob "
            for (int i = 0; i < pla->num_outputs; i++) {
                sscanf(label_start, "%s", pla->outputs[i]);
                label_start += strlen(pla->outputs[i]) + 1; // Move to the next label
            }
        } else {
            // Read the truth table row
            strcpy(pla->table[pla->row_count++], line);
        }
    }

    fclose(file);
}

// Print the PLA contents for debugging
void print_pla(const PLA *pla) {
    printf("Inputs: %d, Outputs: %d\n", pla->num_inputs, pla->num_outputs);
    printf("Truth Table:\n");
    for (int i = 0; i < pla->row_count; i++) {
        printf("%s", pla->table[i]);
    }
}

// Function to combine terms
int combine_terms(const char *term1, const char *term2, char *result, int num_inputs) {
    int differences = 0;

    // Ensure outputs match (the last character of the term)
    if (term1[num_inputs] != term2[num_inputs]) return 0;

    // Now compare inputs, allowing one difference
    for (int i = 0; i < num_inputs; i++) {
        if (term1[i] != term2[i]) {
            if (++differences > 1) return 0; // Can't combine if there are more than one difference
            result[i] = '-'; // Indicate don't care condition
        } else {
            result[i] = term1[i];
        }
    }
    result[num_inputs] = term1[num_inputs]; // Copy the output (it's the same)
    result[num_inputs + 1] = '\0';          // Null-terminate the string
    return 1; // Successful combination
}

// Simplify the PLA
void minimize_pla(PLA *pla) {
    for (int i = 0; i < pla->row_count; i++) {
        if (pla->table[i][0] == '\0') continue; // Skip unused terms
        for (int j = i + 1; j < pla->row_count; j++) {
            if (pla->table[j][0] == '\0') continue; // Skip unused terms
            char combined[MAX_INPUTS + 1];
            if (combine_terms(pla->table[i], pla->table[j], combined, pla->num_inputs)) {
                printf("Combining %s and %s -> %s\n", pla->table[i], pla->table[j], combined);
                strcpy(pla->table[i], combined);   // Replace the first term
                pla->table[j][0] = '\0'; // Mark the second term as unused
                pla->row_count--; // Decrease row count as we have combined two terms
            }
        }
    }
}

// Save the minimized PLA
void save_minimized_pla(const char *file_name, const PLA *pla) {
    FILE *file = fopen(file_name, "w");
    if (!file) {
        perror("Error opening file for saving");
        exit(EXIT_FAILURE);
    }

    // Write the header
    fprintf(file, ".i %d\n", pla->num_inputs);
    fprintf(file, ".o %d\n", pla->num_outputs);

    // Write the input labels
    fprintf(file, ".ilb ");
    for (int i = 0; i < pla->num_inputs; i++) {
        fprintf(file, "%s ", pla->inputs[i]);
    }
    fprintf(file, "\n");

    // Write the output labels
    fprintf(file, ".ob ");
    for (int i = 0; i < pla->num_outputs; i++) {
        fprintf(file, "%s ", pla->outputs[i]);
    }
    fprintf(file, "\n");

    // Write the minimized truth table
    for (int i = 0; i < pla->row_count; i++) {
        if (pla->table[i][0] != '\0') { // Skip unused terms
            fprintf(file, "%s\n", pla->table[i]);
        }
    }

    fclose(file);
}

int main() {
    PLA pla = {0};
    const char *input_file = "espresso.pla";
    const char *output_file = "output.pla";

    parse_pla(input_file, &pla);
    printf("Original PLA:\n");
    print_pla(&pla);

    printf("\nMinimizing the PLA...\n");
    minimize_pla(&pla);

    printf("\nSaving the minimized PLA...\n");
    save_minimized_pla(output_file, &pla);
    printf("Output PLA saved to %s\n", output_file);

    return 0;
}
