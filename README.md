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
### 1.1 Compilation and Execution

**To compile (on terminal)**:

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
    int N = 0;
    double a = 0.0;  // inizializzata per evitare warning

    FILE *fconf = fopen("config.txt", "r");
    if (fconf == NULL) {
        printf("Errore: impossibile aprire config.txt\n");
        return 1;
    }

    char riga[256], valore[128];

    // Parsing del file di configurazione
    while (fgets(riga, sizeof(riga), fconf)) {
        if (strstr(riga, "file_x")) {
            leggi_valore(riga, file_x);
        } else if (strstr(riga, "file_y")) {
            leggi_valore(riga, file_y);
        } else if (strstr(riga, "N")) {
            leggi_valore(riga, valore);
            N = atoi(valore);
        } else if (strstr(riga, "a")) {
            leggi_valore(riga, valore);
            a = atof(valore);
        } else if (strstr(riga, "prefix_output")) {
            leggi_valore(riga, prefix_output);
        }
    }

    fclose(fconf);

    // Controlli basilari sui parametri letti
    if (N <= 0) {
        printf("Errore: N non valido (%d)\n", N);
        return 1;
    }

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
    char filename_d[256];  // buffer più grande
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
### 2.1 Compilation and Execution

**To compile (on terminal)**:

gcc compute_ax_plus_y.c -o compute_ax_plus_y


**To run the program**

./compute_ax_plus_y

**Outcome from the terminal:**

File x: vector_N10_x.dat

File y: vector_N10_y.dat

N: 10

a: 3.00

Output prefix: vector_

Operazione completata. Risultato salvato in vector_N10_d.dat

**Outcome from the file vector_N10_d.dat**

7.40, 7.40, 7.40, 7.40, 7.40, 7.40, 7.40, 7.40, 7.40, 7.40 

## 3 Makefile

To compile both programs (`generate_vectors.c` and `compute_ax_plus_y.c`) we can use the following `Makefile`:

```makefile
CC     = gcc
CFLAGS = -Wall -Wextra -O2

# Main target: compile both programs
all: generate_vectors compute_ax_plus_y

# Compile generate_vectors
generate_vectors: generate_vectors.c
	$(CC) $(CFLAGS) generate_vectors.c -o generate_vectors

# Compile compute_ax_plus_y
compute_ax_plus_y: compute_ax_plus_y.c
	$(CC) $(CFLAGS) compute_ax_plus_y.c -o compute_ax_plus_y

# Clean all generated files
clean:
	rm -f generate_vectors compute_ax_plus_y *.o vector_*.dat
```

## 4 Code for HDF5 files:

Before modify our code to use and read hdf5 files, we have to update the Makefile for the points 4 and 5 of the task03:

```makefile
CC       = gcc
CFLAGS   = -Wall -Wextra -O2
GSL_LIBS = -lgsl -lgslcblas -lm
HDF5_LIBS = -lhdf5

all: generate_vectors compute_ax_plus_y compute_ax_plus_y_gsl \
     generate_vectors_hdf5 compute_ax_plus_y_hdf5

generate_vectors: generate_vectors.c
	$(CC) $(CFLAGS) generate_vectors.c -o generate_vectors

compute_ax_plus_y: compute_ax_plus_y.c
	$(CC) $(CFLAGS) compute_ax_plus_y.c -o compute_ax_plus_y

compute_ax_plus_y_gsl: compute_ax_plus_y_gsl.c
	$(CC) $(CFLAGS) compute_ax_plus_y_gsl.c -o compute_ax_plus_y_gsl $(GSL_LIBS)

generate_vectors_hdf5: generate_vectors_hdf5.c
	$(CC) $(CFLAGS) generate_vectors_hdf5.c -o generate_vectors_hdf5 $(HDF5_LIBS)

compute_ax_plus_y_hdf5: compute_ax_plus_y_hdf5.c
	$(CC) $(CFLAGS) compute_ax_plus_y_hdf5.c -o compute_ax_plus_y_hdf5 $(HDF5_LIBS)

clean:
	rm -f generate_vectors compute_ax_plus_y compute_ax_plus_y_gsl \
	      generate_vectors_hdf5 compute_ax_plus_y_hdf5 \
	      *.o vector_*.dat *.h5
```

### 4.1 generate vectors in hdf5 file format

```c
#include <stdio.h>
#include <stdlib.h>
#include "hdf5.h"

int main(int argc, char *argv[]) {
    if (argc < 3) {
        printf("Usage: %s <N> <output_hdf5_file>\n", argv[0]);
        return 1;
    }

    int N = atoi(argv[1]);
    const char *filename = argv[2];

    if (N <= 0) {
        printf("Error: N must be > 0\n");
        return 1;
    }

    // Alloca i vettori in memoria
    double *x = malloc(N * sizeof(double));
    double *y = malloc(N * sizeof(double));

    if (x == NULL || y == NULL) {
        perror("Error allocating memory");
        return 1;
    }

    // Riempie i vettori
    for (int i = 0; i < N; i++) {
        x[i] = 0.1;
        y[i] = 7.1;
    }

    // --- Scrittura in HDF5 ---

    // Crea il file HDF5 (sovrascrive se esiste)
    hid_t file_id = H5Fcreate(filename, H5F_ACC_TRUNC,
                              H5P_DEFAULT, H5P_DEFAULT);
    if (file_id < 0) {
        printf("Error creating HDF5 file %s\n", filename);
        free(x);
        free(y);
        return 1;
    }

    // Spazio dati 1D di dimensione N
    hsize_t dims[1] = { (hsize_t) N };
    hid_t dataspace_id = H5Screate_simple(1, dims, NULL);

    if (dataspace_id < 0) {
        printf("Error creating dataspace\n");
        H5Fclose(file_id);
        free(x);
        free(y);
        return 1;
    }

    // Crea dataset "x"
    hid_t dset_x_id = H5Dcreate(file_id, "/x",
                                H5T_NATIVE_DOUBLE, dataspace_id,
                                H5P_DEFAULT, H5P_DEFAULT, H5P_DEFAULT);
    if (dset_x_id < 0) {
        printf("Error creating dataset x\n");
        H5Sclose(dataspace_id);
        H5Fclose(file_id);
        free(x);
        free(y);
        return 1;
    }

    // Scrive il vettore x nel dataset "x"
    if (H5Dwrite(dset_x_id, H5T_NATIVE_DOUBLE,
                 H5S_ALL, H5S_ALL, H5P_DEFAULT, x) < 0) {
        printf("Error writing dataset x\n");
        H5Dclose(dset_x_id);
        H5Sclose(dataspace_id);
        H5Fclose(file_id);
        free(x);
        free(y);
        return 1;
    }

    H5Dclose(dset_x_id);

    // Crea dataset "y"
    hid_t dset_y_id = H5Dcreate(file_id, "/y",
                                H5T_NATIVE_DOUBLE, dataspace_id,
                                H5P_DEFAULT, H5P_DEFAULT, H5P_DEFAULT);
    if (dset_y_id < 0) {
        printf("Error creating dataset y\n");
        H5Sclose(dataspace_id);
        H5Fclose(file_id);
        free(x);
        free(y);
        return 1;
    }

    // Scrive il vettore y nel dataset "y"
    if (H5Dwrite(dset_y_id, H5T_NATIVE_DOUBLE,
                 H5S_ALL, H5S_ALL, H5P_DEFAULT, y) < 0) {
        printf("Error writing dataset y\n");
        H5Dclose(dset_y_id);
        H5Sclose(dataspace_id);
        H5Fclose(file_id);
        free(x);
        free(y);
        return 1;
    }

    H5Dclose(dset_y_id);
    H5Sclose(dataspace_id);
    H5Fclose(file_id);

    free(x);
    free(y);

    printf("HDF5 file generated: %s (datasets: /x, /y, size N = %d)\n",
           filename, N);
    return 0;
}

```
### 4.2 read info from HDF5 file and perform d=ax+y

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "hdf5.h"

// Funzione per estrarre il valore dopo '=' e togliere newline
void leggi_valore(char *riga, char *valore) {
    char *uguale = strchr(riga, '=');
    if (uguale != NULL) {
        strcpy(valore, uguale + 1);
        // Rimuove newline
        valore[strcspn(valore, "\r\n")] = '\0';
        // Rimuove spazi iniziali
        while (*valore == ' ') valore++;
    }
}

int main(void) {
    char file_h5[128], dataset_x[128], dataset_y[128], prefix_output[128];
    int N = 0;
    double a = 0.0;

    FILE *fconf = fopen("config_hdf5.txt", "r");
    if (fconf == NULL) {
        printf("Error: cannot open config_hdf5.txt\n");
        return 1;
    }

    char riga[256], valore[128];

    // Parsing del file di configurazione
    while (fgets(riga, sizeof(riga), fconf)) {
        if (strstr(riga, "file_h5")) {
            leggi_valore(riga, file_h5);
        } else if (strstr(riga, "dataset_x")) {
            leggi_valore(riga, dataset_x);
        } else if (strstr(riga, "dataset_y")) {
            leggi_valore(riga, dataset_y);
        } else if (strstr(riga, "N")) {
            leggi_valore(riga, valore);
            N = atoi(valore);
        } else if (strstr(riga, "a")) {
            leggi_valore(riga, valore);
            a = atof(valore);
        } else if (strstr(riga, "prefix_output")) {
            leggi_valore(riga, prefix_output);
        }
    }

    fclose(fconf);

    if (N <= 0) {
        printf("Error: invalid N (%d)\n", N);
        return 1;
    }

    printf("HDF5 input file : %s\n", file_h5);
    printf("Dataset x       : %s\n", dataset_x);
    printf("Dataset y       : %s\n", dataset_y);
    printf("N               : %d\n", N);
    printf("a               : %.2f\n", a);
    printf("Output prefix   : %s\n", prefix_output);

    // Alloca i vettori
    double *x = malloc(N * sizeof(double));
    double *y = malloc(N * sizeof(double));
    double *d = malloc(N * sizeof(double));

    if (x == NULL || y == NULL || d == NULL) {
        printf("Error allocating memory.\n");
        return 1;
    }

    // --- Lettura da HDF5 ---

    // Apre il file HDF5 in sola lettura
    hid_t file_id = H5Fopen(file_h5, H5F_ACC_RDONLY, H5P_DEFAULT);
    if (file_id < 0) {
        printf("Error opening HDF5 file %s\n", file_h5);
        free(x); free(y); free(d);
        return 1;
    }

    // Apre il dataset x
    hid_t dset_x_id = H5Dopen(file_id, dataset_x, H5P_DEFAULT);
    if (dset_x_id < 0) {
        printf("Error opening dataset %s\n", dataset_x);
        H5Fclose(file_id);
        free(x); free(y); free(d);
        return 1;
    }

    // Legge x
    if (H5Dread(dset_x_id, H5T_NATIVE_DOUBLE,
                H5S_ALL, H5S_ALL, H5P_DEFAULT, x) < 0) {
        printf("Error reading dataset %s\n", dataset_x);
        H5Dclose(dset_x_id);
        H5Fclose(file_id);
        free(x); free(y); free(d);
        return 1;
    }
    H5Dclose(dset_x_id);

    // Apre il dataset y
    hid_t dset_y_id = H5Dopen(file_id, dataset_y, H5P_DEFAULT);
    if (dset_y_id < 0) {
        printf("Error opening dataset %s\n", dataset_y);
        H5Fclose(file_id);
        free(x); free(y); free(d);
        return 1;
    }

    // Legge y
    if (H5Dread(dset_y_id, H5T_NATIVE_DOUBLE,
                H5S_ALL, H5S_ALL, H5P_DEFAULT, y) < 0) {
        printf("Error reading dataset %s\n", dataset_y);
        H5Dclose(dset_y_id);
        H5Fclose(file_id);
        free(x); free(y); free(d);
        return 1;
    }
    H5Dclose(dset_y_id);

    H5Fclose(file_id);

    // --- Calcolo d = a*x + y ---
    for (int i = 0; i < N; i++) {
        d[i] = a * x[i] + y[i];
    }

    // --- Scrittura del risultato in HDF5 ---
    char filename_d[256];
    snprintf(filename_d, sizeof(filename_d), "%sN%d_d_hdf5.h5", prefix_output, N);

    hid_t file_out_id = H5Fcreate(filename_d, H5F_ACC_TRUNC,
                                  H5P_DEFAULT, H5P_DEFAULT);
    if (file_out_id < 0) {
        printf("Error creating output HDF5 file %s\n", filename_d);
        free(x); free(y); free(d);
        return 1;
    }

    hsize_t dims[1] = { (hsize_t) N };
    hid_t dataspace_id = H5Screate_simple(1, dims, NULL);

    if (dataspace_id < 0) {
        printf("Error creating dataspace for output\n");
        H5Fclose(file_out_id);
        free(x); free(y); free(d);
        return 1;
    }

    hid_t dset_d_id = H5Dcreate(file_out_id, "/d",
                                H5T_NATIVE_DOUBLE, dataspace_id,
                                H5P_DEFAULT, H5P_DEFAULT, H5P_DEFAULT);
    if (dset_d_id < 0) {
        printf("Error creating dataset /d\n");
        H5Sclose(dataspace_id);
        H5Fclose(file_out_id);
        free(x); free(y); free(d);
        return 1;
    }

    if (H5Dwrite(dset_d_id, H5T_NATIVE_DOUBLE,
                 H5S_ALL, H5S_ALL, H5P_DEFAULT, d) < 0) {
        printf("Error writing dataset /d\n");
        H5Dclose(dset_d_id);
        H5Sclose(dataspace_id);
        H5Fclose(file_out_id);
        free(x); free(y); free(d);
        return 1;
    }

    H5Dclose(dset_d_id);
    H5Sclose(dataspace_id);
    H5Fclose(file_out_id);

    printf("Operation completed (HDF5). Result saved in %s (dataset /d)\n",
           filename_d);

    free(x);
    free(y);
    free(d);

    return 0;
}


```
### 4.3 To run the code we need a config file (.txt)

We can write it like this: 

```file_h5=vectors_N10.h5
dataset_x=/x
dataset_y=/y
N=10
a=3.0
prefix_output=vector_
```

Than we can generate our hdf5 file, from terminal:
```
./generate_vectors_hdf5 10 vectors_N10.h5
```
Now, to calculate d=ax+y:

```
./compute_ax_plus_y_hdf5

```

and to see the result:
```
h5dump -d /d vector_N10_d_hdf5.h5

```

## 5 Code for gsl_vector_axpby:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <gsl/gsl_vector.h>

// Funzione per estrarre il valore dopo '='
void leggi_valore(char *riga, char *valore) {
    char *uguale = strchr(riga, '=');
    if (uguale != NULL) {
        strcpy(valore, uguale + 1);
        valore[strcspn(valore, "\r\n")] = '\0';  // rimuove \n o \r\n
        while (*valore == ' ') valore++;         // rimuove spazi iniziali
    }
}

int main(void) {
    char file_x[128], file_y[128], prefix_output[128];
    int N = 0;
    double a = 0.0;

    FILE *fconf = fopen("config.txt", "r");
    if (fconf == NULL) {
        printf("Errore: impossibile aprire config.txt\n");
        return 1;
    }

    char riga[256], valore[128];

    // Parsing del file di configurazione
    while (fgets(riga, sizeof(riga), fconf)) {
        if (strstr(riga, "file_x")) {
            leggi_valore(riga, file_x);
        } else if (strstr(riga, "file_y")) {
            leggi_valore(riga, file_y);
        } else if (strstr(riga, "N")) {
            leggi_valore(riga, valore);
            N = atoi(valore);
        } else if (strstr(riga, "a")) {
            leggi_valore(riga, valore);
            a = atof(valore);
        } else if (strstr(riga, "prefix_output")) {
            leggi_valore(riga, prefix_output);
        }
    }

    fclose(fconf);

    if (N <= 0) {
        printf("Errore: N non valido (%d)\n", N);
        return 1;
    }

    printf("File x: %s\n", file_x);
    printf("File y: %s\n", file_y);
    printf("N: %d\n", N);
    printf("a: %.2f\n", a);
    printf("Output prefix: %s\n", prefix_output);

    // Alloca vettori GSL
    gsl_vector *vx = gsl_vector_alloc(N);
    gsl_vector *vy = gsl_vector_alloc(N);

    if (vx == NULL || vy == NULL) {
        printf("Errore allocazione vettori GSL.\n");
        return 1;
    }

    // Apre i file di input
    FILE *fx = fopen(file_x, "r");
    FILE *fy = fopen(file_y, "r");

    if (fx == NULL || fy == NULL) {
        printf("Errore apertura file x o y.\n");
        gsl_vector_free(vx);
        gsl_vector_free(vy);
        return 1;
    }

    // Legge i dati e li inserisce nei vettori GSL
    for (int i = 0; i < N; i++) {
        double valx, valy;
        if (fscanf(fx, "%lf", &valx) != 1) {
            printf("Errore lettura x[%d]\n", i);
            fclose(fx);
            fclose(fy);
            gsl_vector_free(vx);
            gsl_vector_free(vy);
            return 1;
        }
        if (fscanf(fy, "%lf", &valy) != 1) {
            printf("Errore lettura y[%d]\n", i);
            fclose(fx);
            fclose(fy);
            gsl_vector_free(vx);
            gsl_vector_free(vy);
            return 1;
        }
        gsl_vector_set(vx, i, valx);
        gsl_vector_set(vy, i, valy);
    }

    fclose(fx);
    fclose(fy);

    // Calcolo con GSL: vy = a*vx + 1.0*vy  → vy contiene d = a*x + y
    gsl_vector_axpby(a, vx, 1.0, vy);

    // Costruzione del nome del file di output
    char filename_d[256];
    snprintf(filename_d, sizeof(filename_d), "%sN%d_d_gsl.dat", prefix_output, N);

    FILE *fd = fopen(filename_d, "w");
    if (fd == NULL) {
        printf("Errore apertura file di output.\n");
        gsl_vector_free(vx);
        gsl_vector_free(vy);
        return 1;
    }

    for (int i = 0; i < N; i++) {
        double di = gsl_vector_get(vy, i);
        fprintf(fd, "%.2f\n", di);
    }

    fclose(fd);

    gsl_vector_free(vx);
    gsl_vector_free(vy);

    printf("Operazione completata (GSL). Risultato salvato in %s\n", filename_d);
    return 0;
}
```
