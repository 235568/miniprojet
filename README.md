Ce code implémente une version simplifiée de l'AES-128 (128 bits). Pour un usage réel, des optimisations et des vérifications
supplémentaires seraient nécessaires pour s'assurer que le code est conforme aux spécifications de l'AES et sécurisé.
Ce code inclut les principales étapes de l'AES : SubBytes, ShiftRows, MixColumns, et AddRoundKey.

 #le code : 
 
 library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

entity aes is
  port (
    clk          : in  std_logic;
    reset        : in  std_logic;
    data_in      : in  std_logic_vector(127 downto 0);
    key          : in  std_logic_vector(127 downto 0);
    start        : in  std_logic;
    data_out     : out std_logic_vector(127 downto 0);
    done         : out std_logic
  );
end aes;

architecture Behavioral of aes is

  -- AES S-Box
  function SubBytes (input : std_logic_vector(7 downto 0)) return std_logic_vector is
    variable sbox : std_logic_vector(7 downto 0);
  begin
    case input is
      when "00000000" => sbox := "11001100";
      when "00000001" => sbox := "11001111";
      -- Les autres valeurs de la S-Box doivent être ajoutées ici
      when others     => sbox := (others => '0');
    end case;
    return sbox;
  end function;

  -- ShiftRows transformation
  function ShiftRows (state : std_logic_vector(127 downto 0)) return std_logic_vector is
    variable temp : std_logic_vector(127 downto 0);
  begin
    temp(127 downto 120) := state(127 downto 120);
    temp(119 downto 112) := state(87 downto 80);
    temp(111 downto 104) := state(47 downto 40);
    temp(103 downto 96)  := state(7 downto 0);
    temp(95 downto 88)   := state(95 downto 88);
    temp(87 downto 80)   := state(55 downto 48);
    temp(79 downto 72)   := state(15 downto 8);
    temp(71 downto 64)   := state(103 downto 96);
    temp(63 downto 56)   := state(63 downto 56);
    temp(55 downto 48)   := state(23 downto 16);
    temp(47 downto 40)   := state(111 downto 104);
    temp(39 downto 32)   := state(71 downto 64);
    temp(31 downto 24)   := state(31 downto 24);
    temp(23 downto 16)   := state(119 downto 112);
    temp(15 downto 8)    := state(79 downto 72);
    temp(7 downto 0)     := state(39 downto 32);
    return temp;
  end function;

  -- AddRoundKey transformation
  function AddRoundKey (state, round_key : std_logic_vector(127 downto 0)) return std_logic_vector is
    variable temp : std_logic_vector(127 downto 0);
  begin
    temp := state xor round_key;
    return temp;
  end function;

  -- AES round function
  function aes_round (state, round_key : std_logic_vector(127 downto 0)) return std_logic_vector is
    variable temp : std_logic_vector(127 downto 0);
    variable i : integer;
  begin
    -- SubBytes
    for i in 0 to 15 loop
      temp(127 - 8 * i downto 120 - 8 * i) := SubBytes(state(127 - 8 * i downto 120 - 8 * i));
    end loop;
    -- ShiftRows
    temp := ShiftRows(temp);
    -- AddRoundKey
    temp := AddRoundKey(temp, round_key);
    return temp;
  end function;

  signal state_reg : std_logic_vector(127 downto 0);
  signal round_key : std_logic_vector(127 downto 0);
  signal round     : integer := 0;
  signal busy      : std_logic := '0';

begin

  process (clk, reset)
  begin
    if reset = '1' then
      state_reg <= (others => '0');
      round_key <= (others => '0');
      round     <= 0;
      busy      <= '0';
      done      <= '0';
    elsif rising_edge(clk) then
      if start = '1' and busy = '0' then
        state_reg <= data_in;
        round_key <= key;
        round     <= 0;
        busy      <= '1';
        done      <= '0';
      elsif busy = '1' then
        if round < 10 then
          state_reg <= aes_round(state_reg, round_key);
          round_key <= round_key;  -- Clé suivante (expansion de clé non implémentée)
          round     <= round + 1;
        else
          busy <= '0';
          done <= '1';
        end if;
      end if;
    end if;
  end process;

  data_out <= state_reg;

end Behavioral;



#Explications des étapes principales :

SubBytes : Cette fonction effectue une substitution non linéaire de chaque octet de l'état en utilisant une S-Box pré-définie. 
La S-Box complète doit être ajoutée pour une implémentation complète.

ShiftRows : Cette fonction réorganise les octets de l'état de manière spécifique, décalant les rangées par différents offsets.

AddRoundKey : Cette fonction applique un XOR entre l'état et la clé de ronde.

aes_round : Cette fonction regroupe les transformations SubBytes, ShiftRows et AddRoundKey pour une ronde d'AES.

Processus principal : Le processus séquentiel qui gère les cycles de rondes de l'AES. Il démarre avec l'entrée data_in et la clé initiale key, 
puis applique les rondes nécessaires jusqu'à ce que l'opération soit terminée.

# Notes importantes :
S-Box : La S-Box fournie est incomplète et doit être complétée pour toutes les valeurs possibles de 8 bits.
Expansion de clé : L'expansion de clé n'est pas implémentée dans cet exemple. En AES-128, l'expansion de clé génère 11 clés de 128 bits à partir de la clé initiale.
MixColumns : Cette transformation n'est pas incluse dans cet exemple, mais elle est essentielle pour une implémentation complète de l'AES.
Pour un usage dans un contexte de sécurité réel, il est crucial de s'assurer que toutes les étapes de l'AES sont correctement implémentées
et que le code est sécurisé contre les attaques possibles.

