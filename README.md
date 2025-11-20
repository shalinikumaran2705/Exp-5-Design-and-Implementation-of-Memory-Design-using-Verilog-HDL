# Exp-5-Design-and-Simulate the-Memory-Design-using-Verilog-HDL
#Aim
To design and simulate a RAM,ROM,FIFO using Verilog HDL, and verify its functionality through a testbench in the Vivado 2023.1 environment.
Apparatus Required
Vivado 2023.1
Procedure
1. Launch Vivado 2023.1
Open Vivado and create a new project.
2. Design the Verilog Code
Write the Verilog code for the RAM,ROM,FIFO
3. Create the Testbench
Write a testbench to simulate the memory behavior. The testbench should apply various and monitor the corresponding output.
4. Create the Verilog Files
Create both the design module and the testbench in the Vivado project.
5. Run Simulation
Run the behavioral simulation to verify the output.
6. Observe the Waveforms
Analyze the output waveforms in the simulation window, and verify that the correct read and write operation.
7. Save and Document Results
Capture screenshots of the waveform and save the simulation logs. These will be included in the lab report.

# RAM
# Code
```
module ram (
    input clk,
    input rst,
    input en,
    input [7:0] datain,
    input [9:0] address,
    output reg [7:0] dataout
);

    reg [7:0] mem [1023:0];

    always @(posedge clk) begin
        if (rst) begin
            dataout <= 8'b0;
        end
        else if (en) begin
            mem[address] <= datain;
        end
        else begin
            dataout <= mem[address];
        end
    end
endmodule
```
# TestBench

```
`timescale 1ns/1ps
module ram_tb;

    reg clk;
    reg rst;
    reg en;
    reg [7:0] datain;
    reg [9:0] address;
    wire [7:0] dataout;

    ram DUT (
        .clk(clk),
        .rst(rst),
        .en(en),
        .datain(datain),
        .address(address),
        .dataout(dataout)
    );

    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    initial begin
        rst = 1;
        en = 0;
        datain = 0;
        address = 0;
        #10;

        rst = 0;
        en = 1;
        address = 10'd5;
        datain = 8'd25;
        #10;

        en = 1;
        address = 10'd10;
        datain = 8'd50;
        #10;

        en = 0;
        address = 10'd5;
        #10;

        address = 10'd10;
        #10;

        $finish;
    end

endmodule
```

# Output

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/79ceb587-9742-45aa-a2b4-da1655d9f0c7" />


# ROM
# Code

 ```
module rom (
   input clk,
   input [9:0] address,
   output reg [7:0] dataout
);

   reg [7:0] mem [1023:0];
   integer i;

   initial begin
       for(i = 0; i < 1024; i = i + 1)
           mem[i] = $random;
   end

   always @(posedge clk) begin
       dataout <= mem[address];
   end
endmodule

```
 # Testbench
 ```
`timescale 1ns/1ps
module rom_tb;

   reg clk;
   reg [9:0] address;
   wire [7:0] dataout;

   rom DUT (
       .clk(clk),
       .address(address),
       .dataout(dataout)
   );

   initial begin
       clk = 0;
       forever #5 clk = ~clk;
   end

   initial begin
       address = 10'd0;
       #10;

       address = 10'd1;
       #10;

       address = 10'd2;
       #10;

       address = 10'd3;
       #10;

       address = 10'd4;
       #10;

       $finish;
   end
endmodule
```

# Output

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/fca88061-c4f0-49ce-ba93-79763b7fb388" />


 # FIFO
# Code
 ```
`timescale 1ns / 1ps

module fifo_sync(
    input clk,
    input rst,
    input wr_en,
    input rd_en,
    input [7:0] data_in,
    output reg [7:0] data_out,
    output reg full,
    output reg empty,
    output reg [4:0] count
);

    reg [7:0] mem [0:15];      // 16x8 FIFO memory
    reg [3:0] wr_ptr;          // write pointer
    reg [3:0] rd_ptr;          // read pointer

    always @(posedge clk) begin
        if (rst) begin
            wr_ptr   <= 0;
            rd_ptr   <= 0;
            count    <= 0;
            data_out <= 0;
            full     <= 0;
            empty    <= 1;
        end else begin
            // Write operation
            if (wr_en && !full) begin
                mem[wr_ptr] <= data_in;
                wr_ptr <= wr_ptr + 1'b1;
            end

            // Read operation
            if (rd_en && !empty) begin
                data_out <= mem[rd_ptr];
                rd_ptr <= rd_ptr + 1'b1;
            end

            // Update count
            case ({wr_en && !full, rd_en && !empty})
                2'b10: count <= count + 1'b1; // only write
                2'b01: count <= count - 1'b1; // only read
                default: count <= count;      // both or none
            endcase

            // Update status flags
            full  <= (count == 16);
            empty <= (count == 0);
        end
    end
endmodule
```
 
# Testbench
 ```
module fifo_sync_tb;
    reg clk;
    reg rst;
    reg wr_en;
    reg rd_en;
    reg [7:0] data_in;
    wire [7:0] data_out;
    wire full;
    wire empty;
    wire [4:0] count;

    // Corrected port order (match module declaration)
    fifo_sync uut (
        .clk(clk),
        .rst(rst),
        .wr_en(wr_en),
        .rd_en(rd_en),
        .data_in(data_in),
        .data_out(data_out),
        .full(full),
        .empty(empty),
        .count(count)
    );

    // Clock generation
    always #5 clk = ~clk;

    initial begin
        clk = 0;
        rst = 1;
        wr_en = 0;
        rd_en = 0;
        data_in = 8'h00;

        #10 rst = 0;
        #10 rst = 1;
        #10 rst = 0;

        // Write 5 values
        repeat (5) begin
            @(posedge clk);
            wr_en = 1;
            data_in = data_in + 1;
        end
        @(posedge clk);
        wr_en = 0;

        // Read 3 values
        repeat (3) begin
            @(posedge clk);
            rd_en = 1;
        end
        @(posedge clk);
        rd_en = 0;

        #20;
        $finish;
    end
endmodule


```
# Output

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/6b4f7a23-d7a0-4e1b-ad73-8c616b5f6bef" />



# Conclusion
The RAM, ROM, FIFO memory with read and write operations was designed and successfully simulated using Verilog HDL. The testbench verified both the write and read functionalities by simulating the memory operations and observing the output waveforms. The experiment demonstrates how to implement memory operations in Verilog, effectively modeling both the reading and writing processes.
 
 

