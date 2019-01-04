# Parser-Verilog 

A Fast C++ Header-only Parser for Verilog.

# Get Started with Parser-Verilog 

A [Verilog] is a programming language that is used to describe a 
digital circuit. Below is a circuit written in Verilog.

```Verilog 
module simple (
input1,
input2, 
input3,
out
);

// Primary inputs
input input1;
input input2;
input input3;

wire input1;
wire input2;
wire input3;

// Primary output
output out; 

wire out;

// Wires
wire w1;
wire w2;
wire w3;
wire w4;

// Gates
AND2_X1 g1 (.a(input1), .b(input2), .o(w1));
OR2_X1 g2  (.a(input3), .b(w1), .o(w2));
INV_X2 g3  (.a(w2), .o(w3));
NOR2_X1 g4 (.a(w1), .b(w3), .o(out));

endmodule
```

The following example demonstrates how to use Parser-Verilog to parse a Verilog file.


```cpp
#include <iostream>

#include "verilog_driver.hpp"   // The only include you need

// Define your own parser by inheriting the ParserVerilogInterface
struct MyVerilogParser : public verilog::ParserVerilogInterface {

  virtual ~MyVerilogParser(){}

  // Function that will be called when encountering the top module name.
  void add_module(std::string&& name){
    std::cout << "Module: " << name << '\n';
  }

  // Function that will be called when encountering a port.
  void add_port(verilog::Port&& port) {
    std::cout << "Port: " << port << '\n';
  }  

  // Function that will be called when encountering a net.
  void add_net(verilog::Net&& net) {
    std::cout << "Net: " << net << '\n';
  }  

  // Function that will be called when encountering a assignment statement.
  void add_assignment(verilog::Assignment&& ast) {
    std::cout << "Assignment: " << ast << '\n';
  }  

  // Function that will be called when encountering a gate.
  void add_instance(verilog::Instance&& inst) {
    std::cout << "Instance: " << inst << '\n';
  }
};

int main(){
  MyVerilogParser parser;
  parser.read("verilog_file.v");
  return EXIT_SUCCESS;
}
```

You need a C++ compiler with C++17 support, [GNU Bison] and [Flex] to compile Parser-Verilog.
```bash
~$ flex -o./verilog_lexer.yy.cc parser-verilog/verilog_lexer.l 
~$ bison -d -o verilog_parser.tab.cc parser-verilog/verilog_parser.yy
~$ g++ -std=c++17 -I parser-verilog/ verilog_parser.tab.cc verilog_lexer.yy.cc example/simple_parser.cpp -o simple_parser -lstdc++fs
```

# Compile Tests
## System Requirements
To use Parser-Verilog, you need following libraries:
+ GNU [C++ Compiler G++ v7.2](https://gcc.gnu.org/gcc-7/) (or higher) with C++17 support 
+ [GNU Bison] at least 3.0.4
+ [Flex] at least 2.6.0

Currently Parser-Verilog has been tested to run well on Linux distributions.

## Build through CMake 
We use [CMake](https://cmake.org/) to manage the source and tests. 
We recommend using out-of-source build.

```bash
~$ git clone https://github.com/OpenTimer/Parser-Verilog.git
~$ cd Parser-Verilog
~$ mkdir build
~$ cd build
~$ cmake ../
~$ make 
```

# Use Parser-Verilog 
Parser-Verilog is extremely easy to use and understand. You create your own Verilog parser `struct` or `class` that 
inherits the `VerilogParserInterface` and define member functions to be invoked to process the components in a circuit.

## Create your own Verilog parser 
```cpp 
#include "verilog_driver.hpp"   // The only include you need

// Define your own parser by inheriting the ParserVerilogInterface
struct MyVerilogParser : public verilog::ParserVerilogInterface {

  virtual ~MyVerilogParser(){}

  // Implement below member functions to process different components

  void add_module(std::string&& name){
    // Process the module name
  }
  void add_port(verilog::Port&& port) {
    // Process a port
  }  
  void add_net(verilog::Net&& net) {
    // Process a net
  }
  void add_assignment(verilog::Assignment&& ast) {
    // Process an assignment
  }  
  void add_instance(verilog::Instance&& inst) {
    // Process a gate
  }
};
```
Below are the required member functions in your custom Verilog parser

| Name | Argument | Return | Description |
| ----- |:------------------| :-------------- | :-------------- |
| add_module  | std::string | n/a | invoked when encountering the name of top module |
| add_port    | Port  | n/a |  invoked when encountering a primary input/output of the module |
| add_net     | Net  | n/a | invoked when encountering a net declaration |
| add_assignment | Assignment | n/a | invoked when encountering an assignment statement |
| add_instance | Instance | n/a | invoked when encountering a gate |

## Data Structures
We define a set of `structs` storing the information of different components during parsing and 
we invoke your parser's member functions on those data structures.

### Struct Port 
The struct `Port` stores the information of a primary input/output of the module 

| Name | Type | Description |
| ------------- |:-------------| :--------------|
| name   | std::vector<std::string> | the names of ports   |
| PortDirection   | `enum class` | the direction of the port. The value could be either INPUT, OUTPUT or INOUT.   | 
| ConnectionType | `enum class` | the connection type of the port. The value could be either NONE, WIRE, REG. | 
| beg, end   | `int` | the bitwidth (range) of the port   |


### Struct Net
The struct `Net` stores the information of a net declaration

| Name | Type | Description |
| ------------- |:-------------| :--------------|
| name   | std::vector<std::string> | the names of nets   |
| NetType   | `enum class` | the type of a net. The value could be either NONE, REG, WIRE, WAND, WOR, TRI, TRIOR, TRIAND, SUPPLY0, SUPPLY1 | 
| beg, end   | `int` | the bitwidth (range) of the net   |



### Struct Assignment
The struct `Assignment` stores the information of an assignment statement

| Name | Type | Description |
| ------------- |:-------------| :--------------|
| lhs   | std::vector<std::variant<std::string, NetBit, NetRange>> | the left hand side of the assignment statement |
| rhs   | std::vector<std::variant<std::string, NetBit, NetRange, Constant>> | the right hand side of the assignment statement  |




# License

<img align="right" src="http://opensource.org/trademarks/opensource/OSI-Approved-License-100x137.png">

Parser-Verilog is licensed under the [MIT License](./LICENSE):

Copyright &copy; 2019 [Chun-Xun Lin][Chun-Xun Lin], [Tsung-Wei Huang][Tsung-Wei Huang] and [Martin Wong][Martin Wong]

The University of Illinois at Urbana-Champaign, IL, USA

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.


***
[Tsung-Wei Huang]:     http://web.engr.illinois.edu/~thuang19/
[Chun-Xun Lin]:        https://github.com/clin99
[Martin Wong]:         https://ece.illinois.edu/directory/profile/mdfwong
[Verilog]:             https://en.wikipedia.org/wiki/Verilog 
[GNU Bison]:           https://www.gnu.org/software/bison/
[Flex]:                https://github.com/westes/flex 

