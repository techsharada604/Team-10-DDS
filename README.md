# Memory Game
## Team Details:
<details>
  Semester: 3rd Sem B. Tech. CSE <br />
  Section: S1 <br />
1. Megha Arya            221CS136 meghaarya.221cs136@nitk.edu.in 7338430370 <br />
2. Sharada A Pandit      221CS151 sharadaapandit.221cs151@nitk.edu.in 9916411636 <br />
3. Viren Haresh Kundnani 221CS165 virenhareshkundnani.221cs165@nitk.edu.in 8850778784 <br />
</details>

## Abstract:

<details>
In our project, we'll be designing a sequential logic game, such as a memory game.Users must follow a sequence of LED flashes or button presses, and logic gates can 
control the game's logic. The initial sequence will be displayed with the help of LEDs. The player’s input will be taken using pushbuttons. If the input and the 
sequence matches, the next sequence flashes and the player is asked to either input the current or previous sequence to add to the challenge.
In this project, we present the development of an interactive game system that creates a dynamic and engaging user experience. The primary objective of this project is to  design and implement a digital circuit that displays a randomized sequence of LED patterns and validates user input to assess correctness. It offers a unique blend of hardware design, gaming, and user interaction, making it a compelling and engaging project, allowing for an exciting and variable gameplay. Key components of the project include modules for generating random sequences, controlling LEDs to display patterns, capturing user input, and implementing validation logic.
We were motivated to do this project as in today's world full of stress, gaming is an outlet for people to relax. We are implementing a simple form of this relaxation for people to play while also testing their memory.
</details>
  
## Working:
<details>
The project uses sequential circuits, i.e., D Flip Flops and a comparator circuit for the final output of whether the user was right or wrong. This output is also used as an enabler for the generation of the next sequence, hence if it is 0 the game does not continue. The random sequence is generated by using a Linear Feedback shift register,which generates all binary numbers with decimal values from 1 to 15 in random order. This is achieved by XOR’ing the last 2 bits of the previous sequence, shifting the 4 bits to the right (discarding the 4th) and placing this XOR value as the first bit for the new sequence.The user inputs the bits using buttons and the comparator output triggers the clockcircuit for the next sequence to be generated. The LED’s (2-bit : Either 01 or 10) are displayed using a D Flip Flop using the D and Q ends of the flip flop and the sequence sent to the comparator is decided using four 2:1 Multiplexers, each taking one bit of the current and previous sequences as the 2 inputs and LED output as the enabler. Based on the LED displayed to the user, one set of flip flops is used for generation and another set is used for storing the previous flashed sequence and depending on which one is asked, it is sent to the comparator circuit. The comparator simply consists of 4 XOR gates to output whether the input by the user and the sequence were the same.
</details>
  
## Logisim Circuit Diagram:
<details>


![Logisim-snaps](https://github.com/techsharada604/Team-10-DDS/assets/116255115/1dbfc89d-7f1e-49c4-9c26-f4744857b758)

</details>

## Verilog Code:
<details>


  
```verilog
  // 4bit comp
arator

module comparator_4bit (
  input wire [3:0] input1,
  input wire [3:0] input2,
  output wire equal
);

  assign equal = (input1 == input2);

endmodule

module d_flip_flop (
  input wire D,      // Data input
  input wire CLK,    // Clock input
  output reg Q      // Output
);

  //reg Q;            // Output register

  always @(posedge CLK) begin
    Q <= D;       // Data input is transferred to Q on the rising edge of the clock
  end

endmodule

module lfsr4bit (
  input wire CLK,    // Clock input
  output wire [3:0] random_number  // 4-bit random number
);

  wire feedback;
  wire [3:0] lfsr_output;

  // Instantiate four D flip-flops without reset
  d_flip_flop dff0 (.D(feedback), .CLK(CLK), .Q(lfsr_output[0]));
  d_flip_flop dff1 (.D(lfsr_output[2] ^ lfsr_output[3]), .CLK(CLK), .Q(lfsr_output[1]));
  d_flip_flop dff2 (.D(lfsr_output[1]), .CLK(CLK), .Q(lfsr_output[2]));
  d_flip_flop dff3 (.D(lfsr_output[2]), .CLK(CLK), .Q(lfsr_output[3]));

  // Feedback is XOR of the 3rd and 4th flip-flops
  assign feedback = lfsr_output[2] ^ lfsr_output[3];

  assign random_number = lfsr_output;

endmodule

module lfsr_and_storage (
  input wire CLK,        // Clock input
  output wire [0:3] random_number,  // 4-bit random number from LFSR
  output wire [0:3] stored_sequence  // 4-bit stored sequence
);

  wire [3:0] lfsr_output;
  reg [3:0] storage_output;

  // Instantiate the lfsr4bit module
  lfsr4bit lfsr (
    .CLK(CLK),
    .random_number(lfsr_output)
  );

  // Instantiate 4 flip-flops to store the sequence
  always @(posedge CLK) begin
    storage_output <= lfsr_output;
  end

  assign random_number = lfsr_output;
  assign stored_sequence = storage_output;

endmodule

module counter_1_to_2 (
  input wire CLK,    // Clock input
  output wire [1:0] count  // 2-bit counter output
);

  reg [1:0] counter;  // 2-bit counter

  always @(posedge CLK) begin
    // Increment the counter
    if (counter == 2'b01) begin
      counter <= 2'b10;
    end else begin
      counter <= 2'b01;
    end
  end

  assign count = counter;

endmodule

/*module muxes (
  input wire CLK,       // Clock input
  output wire [1:0] count, // 2-bit counter output
  output wire [0:3] mux_outputs // 4-bit multiplexer outputs
);

  reg [1:0] counter;   // 2-bit counter

  wire enable;        // Selection signal for the multiplexers

  // Instantiate the 2-bit counter module
  counter_1_to_2 counter_inst (
    .CLK(CLK),
    .count(count)
  );

  // Use one bit from the counter as the enable signal
  assign enable = count[0];

  // Instantiate 4 2:1 multiplexers
  assign mux_outputs[0] = (enable) ? stored_sequence[0] : random_number[0];
  assign mux_outputs[1] = (enable) ? stored_sequence[1] : random_number[1];
  assign mux_outputs[2] = (enable) ? stored_sequence[2] : random_number[2];
  assign mux_outputs[3] = (enable) ? stored_sequence[3] : random_number[3];

endmodule*/

module multiplexer_2to1 (
  input wire sel,          // Selection signal
  input wire data0,  // Data input 0
  input wire data1,  // Data input 1
  output wire op // Output
);

  assign op = (sel) ? data1 : data0;

endmodule


module counter_enables_muxes (
  input wire CLK,       // Clock input
  output wire [3:0] mux_outputs, // 4-bit multiplexer outputs
  input wire [3:0] lfsr_output,  // 4-bit LFSR output
  input wire [3:0] stored_sequence // 4-bit stored sequence
);

  wire [1:0] counter;   // 2-bit counter
  wire enable;         // Selection signal for the multiplexers

  // Instantiate the 2-bit counter module without reset
  counter_1_to_2 counter_inst (
    .CLK(CLK),
    .count(counter)
  );

  // Determine enable based on the counter value
  assign enable = (counter == 2'b01);

  // Instantiate 4 4:1 multiplexers
  multiplexer_2to1 mux0 ((enable),(lfsr_output[0]),(stored_sequence[0]),(mux_outputs[0]));
  multiplexer_2to1 mux1 ((enable),(lfsr_output[1]),(stored_sequence[1]),(mux_outputs[1]));
  multiplexer_2to1 mux2 ((enable),(lfsr_output[2]),(stored_sequence[2]),(mux_outputs[2]));
  multiplexer_2to1 mux3 ((enable),(lfsr_output[3]),(stored_sequence[3]),(mux_outputs[3]));

endmodule
```




</details>



