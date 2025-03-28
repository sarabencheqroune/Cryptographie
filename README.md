# Cryptographie

Modus Operandi
-----------------------------------------------------
Un script shell qui à partir d’un tel fichier crée deux fichiers, l’un correspondant à l’entête, l’autre aux données binaires.
-----------------------------------------------------
#!/bin/bash

input="UniKorn.ppm"

header="header.ppm"

data="data.bin"

head -n 4 "$input" > "$header"

header_bytes=$(wc -c < "$header")

tail -c +$((header_bytes + 1)) "$input" > "$data"

Un script shell qui chiffre des fichiers de données binaires
-----------------------------------------------------
#!/bin/bash

key="0123456789abcdef0123456789abcdef"

iv="abcdef9876543210abcdef9876543210"

filesize=$(stat -f%z data.bin)

remainder=$((filesize % 16))

if [ $remainder -ne 0 ]; then
    padding=$((16 - remainder))
    echo "[*] Padding with $padding zero bytes..."
    dd if=/dev/zero bs=1 count=$padding status=none >> data.bin
fi

openssl enc -aes-128-ecb -K "$key" -in data.bin -out data_ecb.bin -nopad -nosalt

openssl enc -aes-128-cbc -K "$key" -iv "$iv" -in data.bin -out data_cbc.bin -nopad -nosalt

Un script shell qui reconstruit des images à partir des données chiffrées
----------------------------------------------------------------------------
#!/bin/bash

cat header.ppm data_ecb.bin > reconstructed_ecb.ppm

cat header.ppm data_cbc.bin > reconstructed_cbc.ppm

open reconstructed_ecb.ppm

open reconstructed_cbc.ppm

Les images construites
-----------------------------------------------------
![reconstructed_cbc](https://github.com/user-attachments/assets/3e5fc808-1459-45ed-8061-204da699b0d7)
![reconstructed_ecb](https://github.com/user-attachments/assets/f41691a8-6996-49d9-8550-35969c896999)

Première image (ECB) : On voit encore le poney et même le texte "Vive la Crypto !!!!", malgré le chiffrement.l’image reste reconnaissable même après le chiffrement. 

Deuxième image (CBC) : on ne voit rien de lisible, juste des blocs de couleurs aléatoires. C’est beaucoup plus sécurisé visuellement.

==> Conclusion : ECB est totalement inadapté au chiffrement d’images, CBC masque les répétitions. Donc si un attaquant récupère une image chiffrée en ECB, il pourrait deviner du contenu visuel, et donc briser la confidentialité.

Qu’en déduisez-vous sur le mode opératoire ECB

On remarque que le mode ECB chiffre chaque bloc de données indépendamment. Si une partie de l'image (par exemple, une région avec une couleur uniforme) se répète plusieurs fois dans l'image, les blocs identiques produiront des chiffrés identiques. Cela peut entraîner des motifs visibles ou répétitifs dans l'image chiffrée, rendant le chiffrement vulnérable à des attaques par analyse de fréquence. En clair, l'ECB conserve une structure identifiable de l'image.

Qu’en déduisez-vous comme technique pour mettre à mal la stéganographie

Le mode ECB peut révéler des informations cachées dans les images, car il ne cache pas les motifs répétitifs et cet exercice est la preuve de ce raisonnement vue que l'image UniKorn.ppm cache un message qui est " vive la crypto !!!!!!". Pour la stéganographie, il est préférable d'utiliser des modes comme CBC ou même GCM (Galois/Counter Mode) qui produisent un chiffrement plus complexe et moins prévisible.

Comparaison des algorithmes de chiffrement
-----------------------------------------------------
Un script shell ou un programme en C et en Python qui permette de mesurer le temps de calcul d’un processus.
-----------------------------------------------------
code en C :

#include <stdio.h>

#include <stdlib.h>

#include <time.h>

int main() {

    clock_t start, end;
    double cpu_time_used;
    start = clock();
    system("openssl enc -aes-128-cbc -in large_file.txt -out encrypted_file.bin -K 0123456789abcdef -iv 0000000000000000");
    end = clock();
    cpu_time_used = ((double)(end - start)) / CLOCKS_PER_SEC;
    printf("Temps d'exécution: %f secondes\n", cpu_time_used);
    return 0;
}

en python :
import time

import subprocess

start_time = time.time()

subprocess.run(["openssl", "enc", "-aes-128-cbc", "-in", "large_file.txt", "-out", "encrypted_file.bin", "-K", "0123456789abcdef", "-iv", "0000000000000000"])

end_time = time.time()

execution_time = end_time - start_time

print(f"Temps d'exécution: {execution_time} secondes")

RESULTAT DE MON CODE :
Temps d'exécution: 0.3978869915008545 secondes

Mesurer le temps de calcul pris par différents cryptosystème
-----------------------------------------------------
#!/bin/bash

input_file="large_file.txt"

output_file="encrypted_file.bin"

key="0123456789abcdef"

iv="0000000000000000"

algorithms=("aes-128-cbc" "aes-128-ecb" "des-ede3-cbc" "bf-cbc")

for algo in "${algorithms[@]}"; do

    echo "Chiffrement avec $algo"
    { time openssl enc -$algo -in $input_file -out $output_file -K $key -iv $iv; } 2>&1 | grep real
    
done

Construire un tableau de records de vitesse et nommer le gagnant de la compétition !
-----------------------------------------------------
[Running] /bin/bash "/var/folders/ld/85qwr0cs4jd5g_bkbt5gsq6m0000gn/T/tempCodeRunnerFile.shellscript"
Chiffrement avec aes-128-cbc

real	0m0.034s

Chiffrement avec aes-128-ecb

real	0m0.027s

Chiffrement avec des-ede3-cbc

real	0m0.011s

Chiffrement avec bf-cbc

real	0m0.006s

[Done] exited with code=0 in 0.098 seconds


==> le gagnant est donc le chiffrement avec bf-cbc
