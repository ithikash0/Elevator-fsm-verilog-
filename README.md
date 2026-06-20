`timescale 1ns/1ps

module tb_advanced_elevator_controller;
   reg clk;
   reg reset;
   reg req1;
   reg req2;
   reg req3;
   reg overload;
   reg emergency;
    
   wire [1:0] current_floor;
   wire door_open;
   wire busy;
   wire alarm;
   wire moving_up;
   wire moving_down;
  // Instantiate DUT
    advanced_elevator_controller uut (
        .clk(clk),
        .reset(reset),
        .req1(req1),
        .req2(req2),
        .req3(req3),
        .overload(overload),
        .emergency(emergency),
        .current_floor(current_floor),
        .door_open(door_open),
        .busy(busy),
        .alarm(alarm),
        .moving_up(moving_up),
        .moving_down(moving_down)
    );
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end
   // Monitor signals
    initial begin
        $monitor("Time=%0t | Floor=%0d | Door=%b | Busy=%b | Alarm=%b | Up=%b | Down=%b",
                 $time,
                 current_floor,
                 door_open,
                 busy,
                 alarm,
                 moving_up,
                 moving_down);
    end
   // Test sequence
    initial begin
     // Initialize inputs
        reset      = 1;
        req1       = 0;
        req2       = 0;
        req3       = 0;
        overload   = 0;
        emergency  = 0;
     // Hold reset
        #50;
        reset = 0;
     // Test 1 : Go to Floor 2
        #20;
        req2 = 1;
        #10;
        req2 = 0;
     // Wait enough time for movement
        #1200000000;
    // Test 2 : Go to Floor 3
        #20;
        req3 = 1;
        #10;
        req3 = 0;
        #1200000000;
        // Test 3 : Return to Floor 1
        #20;
        req1 = 1;
        #10;
        req1 = 0;
        #2500000000;
        // Test 4 : Overload
        overload = 1;
        #100;
        overload = 0;
        // Test 5 : Emergency
        emergency = 1;
        #100;
        emergency = 0;
        // Finish simulation
        #500;
        $finish;

  end

endmodule
