# 4-digit-BCD-Up-Down-Counter-designed-in-Verilog-HDL-
Dual Operating Modes  Manual Mode: Push-button controlled counting Auto Mode: Clock-driven counting at approximately 0.5 Hz, generated using a 26-bit clock divider from a 100 MHz system clock. Bidirectional Counting  Supports increment and decrement operations Counts seamlessly through the full 0000-9999 BCD range with directional control. 

CODE :-
module top(
    input clk,
    input reset,
    input up_btn,
    input down_btn,
    input mode,          // 0 = manual, 1 = auto
    output reg [6:0] seg,
    output reg [3:0] an
);

// ---------------- CLOCK DIVIDER ----------------
reg [25:0] slow_counter = 0;
wire slow_enable;

always @(posedge clk)
    slow_counter <= slow_counter + 1;

// create pulse instead of clock
assign slow_enable = (slow_counter == 26'd50_000_000);


// ---------------- COUNTER ----------------
reg [13:0] count = 0;
reg up_prev = 0, down_prev = 0;
reg direction = 1;

// Edge detection
always @(posedge clk) begin
    up_prev <= up_btn;
    down_prev <= down_btn;
end

// SINGLE ALWAYS BLOCK (FIXED ✅)
always @(posedge clk or posedge reset) begin
    if (reset) begin
        count <= 0;
        direction <= 1;
    end else begin

        // -------- MANUAL MODE --------
        if (mode == 0) begin
            if (up_btn && !up_prev)
                count <= (count == 9999) ? 0 : count + 1;

            else if (down_btn && !down_prev)
                count <= (count == 0) ? 9999 : count - 1;
        end

        // -------- AUTO MODE --------
        else begin
            // change direction
            if (up_btn && !up_prev)
                direction <= 1;
            else if (down_btn && !down_prev)
                direction <= 0;

            // slow counting
            if (slow_enable) begin
                if (direction)
                    count <= (count == 9999) ? 0 : count + 1;
                else
                    count <= (count == 0) ? 9999 : count - 1;
            end
        end
    end
end


// ---------------- DIGIT SPLIT ----------------
reg [3:0] d0, d1, d2, d3;

always @(*) begin
    d0 = count % 10;
    d1 = (count / 10) % 10;
    d2 = (count / 100) % 10;
    d3 = (count / 1000) % 10;
end


// ---------------- DISPLAY ----------------
reg [19:0] refresh_counter = 0;
reg [1:0] sel;
reg [3:0] digit;

always @(posedge clk)
    refresh_counter <= refresh_counter + 1;

always @(*)
    sel = refresh_counter[19:18];

always @(*) begin
    case(sel)
        2'b00: begin an = 4'b1110; digit = d0; end
        2'b01: begin an = 4'b1101; digit = d1; end
        2'b10: begin an = 4'b1011; digit = d2; end
        2'b11: begin an = 4'b0111; digit = d3; end
    endcase
end


// ---------------- 7-SEG ----------------
always @(*) begin
    case(digit)
        4'd0: seg = 7'b1000000;
        4'd1: seg = 7'b1111001;
        4'd2: seg = 7'b0100100;
        4'd3: seg = 7'b0110000;
        4'd4: seg = 7'b0011001;
        4'd5: seg = 7'b0010010;
        4'd6: seg = 7'b0000010;
        4'd7: seg = 7'b1111000;
        4'd8: seg = 7'b0000000;
        4'd9: seg = 7'b0010000;
        default: seg = 7'b1111111;
    endcase
end

endmodule
