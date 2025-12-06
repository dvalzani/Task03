# Task 03 – Basis of programming

## 1 `generate_vectors.c`
```c
#include <stdio.h>
#include <stdlib.h>

// Programma per generare due vettori x e y e salvarli su file
int main(int argc, char *argv[]) {

    if (argc < 3) {
        printf("Uso corretto: %s <N> <prefix_output>\n", argv[0]);
        return 1;
    }

    int N = atoi(argv[1]);        // Dimensione dei vettori
    char *prefix = argv[2];       // Prefisso per i nomi dei file

    double *x = malloc(N * sizeof(double));
    double *y = malloc(N * sizeof(double));

    if (x == NULL || y == NULL) {
        perror("Errore allocazione memoria");
        return 1;
    }

    // Riempimento dei vettori
    for (int i = 0; i < N; i++) {
        x[i] = 0.1;
        y[i] = 7.1;
    }

    // Creazione dei nomi dei file di output
    char filename_x[128], filename_y[128];
    snprintf(filename_x, sizeof(filename_x), "%sN%d_x.dat", prefix, N);
    snprintf(filename_y, sizeof(filename_y), "%sN%d_y.dat", prefix, N);

    // Apertura dei file
    FILE *fx = fopen(filename_x, "w");
    FILE *fy = fopen(filename_y, "w");

    if (fx == NULL || fy == NULL) {
        perror("Errore apertura file");
        return 1;
    }

    // Scrittura nei file
    for (int i = 0; i < N; i++) {
        fprintf(fx, "%.2f\n", x[i]);
        fprintf(fy, "%.2f\n", y[i]);
    }

    fclose(fx);
    fclose(fy);
    free(x);
    free(y);

    printf("File generati:\n  %s\n  %s\n", filename_x, filename_y);
    return 0;
}
```
1.1 Compilation and Execution

**To compile (oh terminal)**:

gcc generate_vectors.c -o generate_vectors


**To run the program**

./generate_vectors 10 ./vector_


**After execution, the following files will be generated:**

./vector_N10_x.dat
./vector_N10_y.dat



## 2 `compute_ax_plus_y.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Funzione per estrarre il valore dopo '='
void leggi_valore(char *riga, char *valore) {
    char *uguale = strchr(riga, '=');
    if (uguale != NULL) {
        strcpy(valore, uguale + 1);
        valore[strcspn(valore, "\r\n")] = '\0';
        while (*valore == ' ') valore++; // Rimuove spazi iniziali
    }
}

int main() {
    char file_x[128], file_y[128], prefix_output[128];
    int N;
    double a;

    FILE *fconf = fopen("config.txt", "r");
    if (fconf == NULL) {
        printf("Errore: impossibile aprire config.txt\n");
        return 1;
    }

    char riga[256], valore[128];

    // Parsing del file di configurazione
    while (fgets(riga, sizeof(riga), fconf)) {
        if (strstr(riga, "file_x")) { leggi_valore(riga, file_x); }
        else if (strstr(riga, "file_y")) { leggi_valore(riga, file_y); }
        else if (strstr(riga, "N")) { leggi_valore(riga, valore); N = atoi(valore); }
        else if (strstr(riga, "a")) { leggi_valore(riga, valore); a = atof(valore); }
        else if (strstr(riga, "prefix_output")) { leggi_valore(riga, prefix_output); }
    }

    fclose(fconf);

    printf("File x: %s\n", file_x);
    printf("File y: %s\n", file_y);
    printf("N: %d\n", N);
    printf("a: %.2f\n", a);
    printf("Output prefix: %s\n", prefix_output);

    // Alloca i vettori
    double *x = malloc(N * sizeof(double));
    double *y = malloc(N * sizeof(double));
    double *d = malloc(N * sizeof(double));

    if (x == NULL || y == NULL || d == NULL) {
        printf("Errore allocazione memoria.\n");
        return 1;
    }

    // Apre i file di input
    FILE *fx = fopen(file_x, "r");
    FILE *fy = fopen(file_y, "r");

    if (fx == NULL || fy == NULL) {
        printf("Errore apertura file x o y.\n");
        return 1;
    }

    // Legge i dati
    for (int i = 0; i < N; i++) {
        fscanf(fx, "%lf", &x[i]);
        fscanf(fy, "%lf", &y[i]);
    }

    fclose(fx);
    fclose(fy);

    // Calcolo del vettore d = a*x + y
    for (int i = 0; i < N; i++) {
        d[i] = a * x[i] + y[i];
    }

    // Costruzione del nome del file di output
    char filename_d[128];
    snprintf(filename_d, sizeof(filename_d), "%sN%d_d.dat", prefix_output, N);

    // Salvataggio del risultato
    FILE *fd = fopen(filename_d, "w");
    if (fd == NULL) {
        printf("Errore apertura file di output.\n");
        return 1;
    }

    for (int i = 0; i < N; i++) {
        fprintf(fd, "%.2f\n", d[i]);
    }

    fclose(fd);

    free(x);
    free(y);
    free(d);

    printf("Operazione completata. Risultato salvato in %s\n", filename_d);
    return 0;
}
```
