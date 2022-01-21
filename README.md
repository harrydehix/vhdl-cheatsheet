# vhdl-cheatsheet
summarizes most important things to recognize about vhdl

# using stuff from librarys
```vhdl
library IEEE;
use IEEE.std_logic_1164.all;      -- Enthält wichtige Grundtypen z.B. std_logic 
use IEEE.std_logic_unsigned.all;  -- Enthält unsigned-Arithmetik
```

# creating an entity
```vhdl
entity multiplexer is
  port(
    a, b: in std_logic;
    s: in std_logic;
    c: out std_logic    -- hier kein Semikolon
  );
end entity;
```

# creating an architecture
```vhdl
architecture multiplexer_logic of multiplexer is
begin
  c <= a when s = '0' else
       b when s = '1';
end multiplexer_logic;
```

# simple assignments
```vhdl
a <= '1';
```

# when else assignments
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

# with select assignments
this is equal to the code above
```vhdl
with b select a <= 
     "11" when "00",
     "10" when "01",
     "01" when "10",
     "00" when "11";
```

# bitwise boolean arithmetic
works using `not`, `or`, `and`, `xor`, `nand`, `nor`, `xnor` and `()`.
all operators have the same presendece (!). so just use parenthesis^^.
e.g.
```vhdl
c <= a or (not b) or (f and (g xor h));
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
big_endian(0) = '1';      -- results in "1000_0000"
little_endian(0) = '1';   -- results in "0000_0001"
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
my_array <= (0 => '1', others => '0');
my_array <= (0 | 3 | 5 => '1', others => '0');
my_array <= (4 downto 1 => '1', others => '0');
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
