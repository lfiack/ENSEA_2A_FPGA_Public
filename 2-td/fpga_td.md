# TD FPGA

## Séance 1 : Tuto modelsim, bases, arithmétique

### Tutoriel Modelsim

#### Installer le simulateur

En TD et en TP, vous utiliserez le logiciel Modelsim pour simuler vos composants VHDL.

> Modelsim n'est pas disponible sur Mac. Vous pouvez vous rabattre sur l'outil libre et opensource ```ghdl```. 
> Un tuto sera fourni un jour, en attendant débrouillez-vous.

La dernière version disponible sans licence est ```20.1.1``` disponible à cette adresse : 

[Modelsim version 20.1.1](https://www.intel.com/content/www/us/en/software-kit/750666/modelsim-intel-fpgas-standard-edition-software-version-20-1-1.html)

L’éditeur de Modelsim (ainsi que celui de Quartus Prime, que nous utiliserons en TP) est nul. 
Vous pouvez installer autre chose, comme VSCode. Dans ce cas, installez aussi une extension pour le vhdl.

#### Création d'un premier composant

1. Créez un nouveau dossier ```1-tuto_modelsim```

> Pas d’espaces, pas d’accents, sur un disque dur (pas dans un Drive, pas sur une clé USB/disque dur externe)

2. Lancer un éditeur comme VSCode
3. Créez un fichier ```composant_nul.vhd```
4. Copiez le code suivant (Pensez à votre prof : respectez l’indentation!)

```vhdl
library ieee;
use ieee.std_logic_1164.all;

entity composant_nul is
    port (
        sw : in std_logic;
        led : out std_logic
    );
end entity composant_nul;

architecture rtl of composant_nul is
begin
    led <= sw;
end architecture rtl;
```

5. Décrivez en une phrase ce que fait le composant ci-dessus.

#### Écriture du test-bench

1. Créez un fichier ```composant_nul_tb.vhd```
2. Copiez-y le code suivant :

```vhdl
library ieee;
use ieee.std_logic_1164.all;

entity composant_nul_tb is
    -- Entity of a test bench is always empty
end entity composant_nul_tb;

architecture tb of composant_nul_tb is
    signal tb_led : std_logic;
    signal tb_sw : std_logic;
begin
    uut: entity work.composant_nul
    port map
    (
        led => tb_led,
        sw => tb_sw
    );

    process
    begin
        tb_sw <= '0';
        wait for 10 ns;
        tb_sw <= '1';
        wait for 10 ns;
        tb_sw <= '0';
        wait for 10 ns;
        tb_sw <= '1';
        wait for 10 ns;
        wait;
    end process;
end architecture tb;
```

3. Expliquez le code
4. Pourquoi le ```process``` finit-il par le mot-clé ```wait``` ?

#### Fichier de simulation

1. Créez un fichier ```composant_nul.do```
2. Copiez-y le code suivant : 
```tcl
quit -sim

vcom composant_nul.vhd
vcom composant_nul_tb.vhd

vsim -c work.composant_nul_tb

# INPUTS
add wave -divider Inputs:
add wave -color yellow uut/sw

# OUTPUTS
add wave -divider Outputs:
add wave uut/led

run -all
```

#### Simulation avec Modelsim

1. Lancez Modelsim. L'interface est assez austère, c'est comme ça, il faudra s'y faire.
2. Dans l'invite de commande, vous pouvez taper ```pwd``` pour savoir dans quel dossier vous vous situez.

![alt text](figures/modelsim_pwd.png)

3. Puis tapez ```cd <chemin/vers/le/dossier>```

![alt text](figures/modelsim_cd.png)

4. Puis enfin, tapez ```do composant_nul.do```. Modelsim devrait maintenant ressembler à ça :

![alt text](figures/modelsim_do.png)

5. Vous pouvez dézoomer pour voir l'intégralité de vos signaux en cliquant sur la loupe bleue :

![alt text](figures/modelsim_zoom.png)

### Encodeur/Décodeur

Pour concevoir des circuits numériques, vous aurez souvent besoin de convertir des signaux d'une représentation à une autre.
Les circuits responsables de ces conversions sont appelés _encodeurs_ ou _décodeurs_, ou plus simplements _codeurs_.

#### Décodeur 2 vers 4

Dans cet exercice, vous allez écrire le code VHDL d'un décodeur 2 vers 4. Voici sa table de vérité :

| in1 | in0 | out3 | out2 | out1 | out0 | 
|:---:|:---:|:----:|:----:|:----:|:----:|
| 0   |   0 |    0 |    0 |    0 |    1 |
| 0   |   1 |    0 |    0 |    1 |    0 |
| 1   |   0 |    0 |    1 |    0 |    0 |
| 1   |   1 |    1 |    0 |    0 |    0 |

En VHDL, il existe beaucoup de manières différentes de décrire un décodeur : 
* ```with ... select```,
* ```when ... else```,
* ```case ... when``` au sein d'un ```process```,
* ```if ... then ... elsif``` au sein d'un ```process```.

1. Créez un fichier decoder.vhd
2. Choisissez une de ces quatre méthodes et écrivez le code d'un décodeur 2 vers 4
3. Simulez-le à l'aide du _test bench_ suivant

```vhdl
library ieee;
use ieee.std_logic_1164.all;

entity decoder_tb is
end entity decoder_tb;

architecture tb of decoder_tb is
    signal tb_data_in : std_logic_vector(1 downto 0);
    signal tb_data_out : std_logic_vector(3 downto 0);
begin
    uut : entity work.decoder
        port map (
            data_in => tb_data_in,
            data_out => tb_data_out
        );

    process
    begin
        tb_data_in <= "00";
        wait for 10 ns;
        tb_data_in <= "01";
        wait for 10 ns;
        tb_data_in <= "10";
        wait for 10 ns;
        tb_data_in <= "11";
        wait for 10 ns;
        tb_data_in <= "UU"; -- Undefined inputs, should lead to undefined outputs
        wait for 10 ns;
        wait;
    end process;
end architecture tb;
```

### Multiplexeur

Le multiplexeur est un autre circuit de base **très** utilisé. Les multiplexeurs servent notamment à réaliser des structures conditionnelles (```if then else```).

1. Écrivez le code VHDL d'un multiplexeur 2 vers 1 dont la taille de bus sera configurable

> Vous aurez à utiliser le mot-clé ```generic```

2. Testez avec un _test bench_

> En plus du ```port map```, il faudra un ```generic map``` pour configurer le composant

### ALU

L'ALU (Arithmetic and Logic Unit) est un composant que l'on retrouve dans la quasi totalité des microprocesseurs.
C'est un composant qui permet de réaliser une opération au choix sur deux opérandes.

Dans cet exercice, votre ALU permettra d'effectuer 8 opérations :
* Addition
* Soustraction
* Multiplication
* Division
* OU logique
* ET logique
* OU-EXCLUSIF
* NON logique (sur une seule opérande)

1. Combien de bits de sélection sont nécessaires ?
2. Proposez un schéma (sur papier!)
3. Écrivez le code VHDL de l'ALU

> Vous utiliserez des signaux de type ```signed```

> Vous aurez besoin de la librairie ```numeric_std```

4. Testez à l'aide d'un _test bench_

## Séance 2 : Circuits séquentiels
1. Bascule D
    * Timings
2. Registres (séries/parallèles)
3. Compteurs
4. FSM

## Séance 3 : Contrôleur HDMI
1. Generic
2. 