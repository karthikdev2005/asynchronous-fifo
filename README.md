asynchronous fifo using gray pointer (CDC: Clock Domain Crossing)

top module
``module top(data_in,data_out,wr_ena,rd_ena,clk,rst,empty,full);
parameter depth=8;
parameter size=8;
input [size-1:0]data_in;
output [size-1:0]data_out;
input wr_ena,rd_ena;
output empty,full;
wire [size-1:0]mem[depth-1:0];
input clk,rst;
wire clk1,clk2;
wire [$clog2(depth):0]rd_ptr1,rd_ptr,wr_ptr1,wr_ptr2,rd_ptr2,wr_ptr,rd_ptr_g,wr_ptr_g;
divider kttt(.clk(clk),.clk1(clk1),.clk2(clk2));
writing dut1(.wr_ena(wr_ena),.clk1(clk1),.full(full),.empty(empty),.wr_ptr(wr_ptr),.rd_ptr(rd_ptr2),.rst(rst));
binary_to_gray dut2(.count(wr_ptr),.gray(wr_ptr_g));
syncroniser dut11(.data_in(wr_ptr_g),.data_out(wr_ptr1),.clk(clk1));
gray_to_binary dut3(.gray(wr_ptr1),.binary(wr_ptr2));
reading dut4(.rd_ena(rd_ena),.clk2(clk2),.full(full),.empty(empty),.wr_ptr(wr_ptr2),.rd_ptr(rd_ptr),.rst(rst));
binary_to_gray dut112(.count(rd_ptr),.gray(rd_ptr_g));
syncroniser dut12(.data_in(rd_ptr_g),.data_out(rd_ptr1),.clk(clk2));
gray_to_binary dut5(.gray(rd_ptr1),.binary(rd_ptr2));
memory dutt(.data_in(data_in),.data_out(data_out),.wr_ena(wr_ena),.rd_ena(rd_ena),.wr_ptr(wr_ptr),.rd_ptr(rd_ptr),.clk1(clk1),.clk2(clk2));
endmodule


``clk divider module // only for fpga purpose
module divider(clk,clk1,clk2);
input clk;
parameter wr=50000000,rd=200000000;
output reg clk1,clk2;
reg [$clog2(rd):0]t;
reg [$clog2(wr):0]k;
initial
begin
clk1<=0;
clk2<=0;
t<=0;
k<=0;
end
always@(posedge clk)
begin
if(k==wr)
begin
clk1<=~clk1;
k=0;
end
else
begin
k<=k+1;
end
if(t==rd)
begin
clk2<=~clk2;
t<=0;
end
else
t<=t+1;
end
endmodule



``writing and full condition

module writing(wr_ena,clk1,full,empty,wr_ptr,rd_ptr,rst);
parameter depth=8;
parameter size=8;
input wr_ena,clk1,empty,rst;
output reg full;
input [$clog2(depth):0]rd_ptr;
output reg [$clog2(depth):0]wr_ptr;
always@(posedge clk1)
begin
if(rst)
begin
full<=1'b0;
wr_ptr<=0;
end
else
begin
if({(~wr_ptr[$clog2(depth)]),rd_ptr[$clog2(depth)-1:0]}==rd_ptr[$clog2(depth):0])
begin
full<=1'b1;
end
else
begin
full<=1'b0;
end
if(wr_ena && wr_ptr!=8)
begin
wr_ptr<=wr_ptr+1;
end
else if(wr_ptr==8)
begin
wr_ptr<=0;
end
end
end
endmodule

``reading and empty condition

module reading(rd_ena,clk2,full,empty,wr_ptr,rd_ptr,rst);
parameter depth=8,size=8;
input rd_ena,clk2,rst;
output reg empty;
input full;
output reg [$clog2(depth):0]rd_ptr;
input [$clog2(depth):0]wr_ptr;
always@(posedge clk2)
begin
if(rst)
begin
rd_ptr<=0;
empty<=1'b1;
end
else
begin
if(rd_ptr==depth)
begin
rd_ptr<=0;
end
else
begin
if(rd_ena )
begin
rd_ptr<=rd_ptr+1;
end
end
if(rd_ptr[$clog2(depth):0]==wr_ptr[$clog2(depth):0])
begin
empty<=1;
end
else
empty<=0;
end
end
endmodule


``binary to gray converter


module binary_to_gray(count,gray);
parameter depth=8;
input [$clog2(depth):0]count;
output [$clog2(depth):0]gray;
assign gray=count^(count>>1);
endmodule

''gray to binary converter

module gray_to_binary #(parameter DEPTH = 8)(
    input  wire [$clog2(DEPTH):0] gray,
    output reg  [$clog2(DEPTH):0] binary
);

integer i;

always @(*) begin
    binary[$clog2(DEPTH)] = gray[$clog2(DEPTH)];
    for (i = $clog2(DEPTH)-1; i >= 0; i = i - 1) begin
        binary[i] = binary[i+1] ^ gray[i];
    end
end

endmodule


''syncroniser

module syncroniser(data_in,data_out,clk);
parameter depth=8;
input clk;
input[$clog2(depth):0]data_in;
output reg[$clog2(depth):0]data_out;
reg[$clog2(depth):0]data_out_reg;
always@(posedge clk)
begin
data_out_reg<=data_in;
data_out<=data_out_reg;
end
endmodule

''memory

module memory(data_in,data_out,wr_ena,rd_ena,wr_ptr,rd_ptr,clk1,clk2);
parameter size=8,depth=8;
input [size-1:0]data_in;
input clk1,clk2;
output reg [size-1:0]data_out;
input wr_ena,rd_ena;
input [$clog2(depth):0] wr_ptr;
reg [size-1:0]mem[0:depth-1];
input [$clog2(depth):0]rd_ptr;
always@(posedge clk1)
begin
if(wr_ena)
begin
mem[wr_ptr]<=data_in;
end
end
always@(posedge clk2)
begin
if(rd_ena)
begin
data_out<=mem[rd_ptr];
end
end
endmodule




