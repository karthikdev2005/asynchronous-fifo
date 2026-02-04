clk divider module 


``module divider(clk,clk1,clk2);
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
