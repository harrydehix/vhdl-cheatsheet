# vhdl-cheatsheet
summarizes most important things to recognize about vhdl

1. [librarys](#librarys)
2. [entities](#entities)
3. [architectures](#architectures)
4. [signals](#signals)
5. [processes](#processes)
6. [assignments](#assignments)
7. [operators](#operators)
8. [datatypes](#datatypes)
9. [ghdl](#ghdl)
10. [gtkwave](#gtkwave)
11. [testing](#testing)

# librarys
```vhdl
library IEEE;
use IEEE.std_logic_1164.all;      -- contains std_logic and std_logic_vector (and more) 
use IEEE.numeric_std.all;         -- contains unsigned and signed arithmetics
```

# entities
```vhdl
entity multiplexer is
  port(
    a, b: in std_logic;
    s: in std_logic;
    c: out std_logic    -- no semicolon here!
  );
end entity multiplexer;
```
- blackbox view defining inputs and outputs

# architectures
```vhdl
architecture multiplexer_logic of multiplexer is
begin
  c <= a when s = '0' else
       b when s = '1';
end multiplexer_logic;
```
- whitebox view defining the logic
- you cannot write to inputs!
- you cannot read from outputs!
- all assignments are processes!
  - all processes happen in parallel!
  - a signal takes its state at the end of a process!
  - that means that assigning a value to a signal in one process won't affect another process using that signal (not in the same "loop")!
- use signals to reuse your output!

## reusing entities
```vhdl
architecture structural of fullAdder is
signal s1, s2, s3: bit;
component halfAdder
  port (
    x, y : IN Bit;
    sum, cout : out bit
  );
end component;
begin
  u1 : halfAdder port map (x, y, s1, s2);
  u2 : halfAdder port map (s1, cin, sum, s3);
  cout <= S2 OR S3;
end structural;
```

# signals
```vhdl
architecture some_random_architecture of some_random_entity is
signal internal_signal: std_logic;
begin
  internal_signal <= a xor internal_signal;
  c <= internal_signal;
end some_random_architecture;
```
- useful for reusing outputs

# processes
```vhdl
process(clk)
begin
  if(rising_edge(clk) then
    a <= '1';
  else
    a <= '0';
  end if;
end
```
- processes are only "called" if the value of at least one signal in the sensitivity list changes
- common assignments (outside a process) are processes too, all processes happen in parallel!
- signals take their states at the end of a process

## if-statements
```vhdl
if (a = '1') or (c > d) then
  statements;
elsif condition then
  statements;
else
  statements;
end if;
```
- elsif and else clauses are optional

## case-statements
```vhdl
case a is
  when "00" => b <= "1000";
  when "01" => b <= "0100";
  when "10" => b <= "0010";
  when "11" => b <= "0001";
  when others => b <= "0000"; -- fallback case
end case;
```

# assignments

## simple assignments
```vhdl
a <= '1';
```

## when else assignments
```vhdl
a <= "11" when b = "00" else
     "10" when b = "01" else
     "01" when b = "10" else
     "00" when b = "11";
```
its also possible to define a fallback value
```vhdl
a <= "11" when b = "00" else
     "10" when b = "01" else
     "01" when b = "10" else
     "00" when b = "11" else
     "00";
```

## with select assignments
this is equal to the code above
```vhdl
with b select a <= 
     "11" when "00",  -- use commas here
     "10" when "01",
     "01" when "10",
     "00" when "11",
     "00" when others; -- fallback value
```

# operators

## arithmetical
`+`, `-`, `/`, `*`, `mod`

## relational
`=`, `<`, `<=`, `>`, `>=`

## logical
`not`, `or`, `and`, `xor`, `nand`, `nor` and `xnor`.
`not` has the highest precendence, all other operators have the same precendence (!). so just use parenthesis^^.

```vhdl
c <= a or not b or (f and (g xor h));
```

## concatentation-operator
`&` is an operator allowing you to concatenate vectors.
```vhdl
signal a: std_logic_vector(3 downto 0);
signal b: std_logic_vector(3 downto 0);
signal c: std_logic_vector(8 downto 0);
(...)
c <= a & b;
```

# datatypes

## simple

### boolean
can be `true` or `false`. used for conditions.

### bit
can be `1` or `0`. represents a signal.

### std_logic
the better bit. can be `0` (logic 0, all fine), `1` (logic 1, all fine), `Z` (high impendence), `-` (don't care), `U` (uninitialised, cannot know), `X` (unknown, can't determine), `L` (weak signal, probably 0), `H` (weak signal, probably 1), `W` (weak signal, can't tell).
useful for detecting errors while simulating.

## vectors / arrays
vectors are array-like one-dimensional collections of signals having the same type. they are indexed by an integer number starting with 0.

### defining ranges
- `downto` corresponds to _little endian_
- `to` correspons to _big endian_

**e.g.**
```vhdl
signal big_endian : std_logic_vector(0 to 7);
signal little_endian : std_logic_vector(7 downto 0);
(...)
big_endian = "0000_0001";     -- results in index 7 having the value '1'
little_endian = "0000_0001";  -- results in index 0 having the value '0'
```

### assigning arrays
#### assigning single elements
```vhdl
my_array(0) <= '1';
```
#### assigning a part of the array
```vhdl
my_array(3 downto 0) <= "0101";
```
#### positional association
```vhdl
my_array <= ('0', '1', '0', '1');
```
#### named association
```vhdl
my_array <= (0 => '1', others => '0');            -- 0000_0001
my_array <= (0 | 3 | 5 => '1', others => '0');    -- 0010_1001
my_array <= (4 downto 1 => '1', others => '0');   -- 0001_1110
```

### creating custom vector types
first you have to define your array type:
```vhdl
type 4bit_vector is array(3 downto 0) of bit;
```
after that you can simple use it:
```vhdl
signal a: 4bit_vector;
```

### std_logic_vector
represents a vector of bits (std_logic). does not support arithmetics.
```vhdl
signal a: std_logic_vector(7 downto 0);
```

### unsigned
represents a vector of bits (std_logic) supporting unsigned arithmetics.
```vhdl
signal a: unsigned(7 downto 0) := 7;  -- default value is 'U' (uninitialised) to every bit
(...)
a <= a + 1;
```

### signed
represents a vector of bits (std_logic) supporting signed arithmetics.
```vhdl
signal a: signed(7 downto 0) := -7;  -- default value is 'U' (uninitialised) to every bit
(...)
a <= a + 1;
```

### integer
represents an integer. similar to signed/unsigned.
but you
- cannot assign binary or hex values
- cannot use bitwise/logical operators
- cannot set the bit count using `(8 downto 0)` or `(0 to 3)`
  - instead you have to use `range value1 to value2`-syntax
- cannot access single bits
```vhdl
signal a_number: integer range -256 to 255;
```

### converting between std_logic_vector, unsigned, signed and integer
![grafik](https://user-images.githubusercontent.com/29947316/150503250-f9886cf0-88f8-4692-9a00-b67a65ed9e9f.png)
[Source](http://atlas.physics.arizona.edu/~kjohns/downloads/vhdl/VHDL_Lang.pdf)

# ghdl

```bash
sudo apt install ghdl
sudo apt install ghdl-gcc
```

# gtkwave


```bash
sudo apt install gtkwave
```

# testing

## compiling files

```bash
ghdl-gcc -a *.vhd
```

## generating the simulator

```bash
ghdl-gcc -e ENTITY_NAME
```
