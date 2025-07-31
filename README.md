#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

// Convert a single hexadecimal character to integer
int hex_char_to_int(char c) {
    if ('0' <= c && c <= '9') return c - '0';
    if ('a' <= c && c <= 'f') return c - 'a' + 10;
    if ('A' <= c && c <= 'F') return c - 'A' + 10;
    return -1;
}

// Parse two hex digits after \x and return the character value
int parse_hex(const char *str, int *consumed) {
    int hi = hex_char_to_int(str[0]);
    int lo = hex_char_to_int(str[1]);
    if (hi == -1 || lo == -1) return -1;
    *consumed = 2;
    return (hi << 4) | lo;
}

// Parse up to 3 octal digits after \0 and return the character value
int parse_octal(const char *str, int *consumed) {
    int val = 0, i = 0;
    while (i < 3 && str[i] >= '0' && str[i] <= '7') {
        val = val * 8 + (str[i] - '0');
        i++;
    }
    *consumed = i;
    return val;
}

// Print string with escape sequence processing
void print_with_escapes(const char *str) {
    while (*str) {
        if (*str == '\\') {
            str++;
            if (*str == 'n') putchar('\n');
            else if (*str == 't') putchar('\t');
            else if (*str == '\\') putchar('\\');
            else if (*str == 'x') {
                int consumed = 0;
                int val = parse_hex(str + 1, &consumed);
                if (val != -1) {
                    putchar(val);
                    str += consumed;
                } else {
                    // Invalid hex sequence, print as-is
                    putchar('\\');
                    putchar('x');
                }
            } else if (*str == '0') {
                int consumed = 0;
                int val = parse_octal(str + 1, &consumed);
                if (consumed > 0) {
                    putchar(val);
                    str += consumed;
                } else {
                    // Invalid octal, print as-is
                    putchar('\\');
                    putchar('0');
                }
            } else {
                // Unknown escape, print as-is
                putchar('\\');
                putchar(*str);
            }
        } else {
            putchar(*str);
        }
        if (*str) str++;
    }
}

// Main function: parse flags and print arguments accordingly
int main(int argc, char *argv[]) {
    int enable_escape = 0;       // -e flag: enable escape sequences
    int disable_escape = 0;      // -E flag: disable escape sequences (default)
    int suppress_newline = 0;    // -n flag: suppress newline
    int i = 1;

    // Parse flags at the beginning (multiple -neE allowed and combined)
    while (i < argc && argv[i][0] == '-' && argv[i][1] != '\0') {
        int j = 1;
        while (argv[i][j]) {
            if (argv[i][j] == 'n') suppress_newline = 1;
            else if (argv[i][j] == 'e') enable_escape = 1;
            else if (argv[i][j] == 'E') {
                enable_escape = 0;
                disable_escape = 1;
            } else {
                // Not a valid flag, stop parsing
                goto done_parsing_flags;
            }
            j++;
        }
        i++;
    }

done_parsing_flags:

    // Print all remaining arguments
    for (int k = i; k < argc; k++) {
        if (enable_escape)
            print_with_escapes(argv[k]);
        else
            fputs(argv[k], stdout);

        if (k < argc - 1)
            putchar(' ');
    }

    // Print newline unless -n flag is set
    if (!suppress_newline)
        putchar('\n');

    return 0;
}
