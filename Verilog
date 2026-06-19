# Elevator-fsm-verilog-
Finite State Machine (FSM) based Elevator Controller implemented in Verilog
module advanced_elevator_controller(

  input clk,
  input reset,
  input req1,
  input req2,
  input req3,
  input overload,
  input emergency,
  output reg [1:0] current_floor,
  output reg door_open,
  output reg busy,
  output reg alarm,
  output reg moving_up,
  output reg moving_down
);
// STATE DEFINITIONS
parameter IDLE       = 3'b000;
parameter MOVE_UP    = 3'b001;
parameter MOVE_DOWN  = 3'b010;
parameter DOOR_OPEN  = 3'b011;
parameter DOOR_CLOSE = 3'b100;
parameter EMERGENCY  = 3'b101;

reg [2:0] state, next_state;
reg [1:0] target_floor;
reg request_pending;

// CLOCK DIVIDER (100 MHz -> 1 second tick)

reg [25:0] clk_counter;
reg slow_tick;

always @(posedge clk or posedge reset)
begin
    if(reset)
    begin
        clk_counter <= 26'd0;
        slow_tick   <= 1'b0;
    end
    else
    begin
        if(clk_counter == 26'd49_999_999)
        begin
            clk_counter <= 26'd0;
            slow_tick   <= 1'b1;
        end
        else
        begin
            clk_counter <= clk_counter + 1'b1;
            slow_tick   <= 1'b0;
        end
    end
end

// STATE REGISTER
always @(posedge clk or posedge reset)
begin
    if(reset)
        state <= IDLE;
    else
        state <= next_state;
end
// REQUEST STORAGE

always @(posedge clk or posedge reset)
begin
    if(reset)
    begin
        target_floor <= 2'd1;
        request_pending <= 1'b0;
    end
    else
    begin
        if(req1)
        begin
            target_floor <= 2'd1;
            request_pending <= 1'b1;
        end
        else if(req2)
        begin
            target_floor <= 2'd2;
            request_pending <= 1'b1;
        end
        else if(req3)
        begin
            target_floor <= 2'd3;
            request_pending <= 1'b1;
        end
        else if(state == DOOR_CLOSE)
        begin
            request_pending <= 1'b0;
        end
    end
end

// NEXT STATE LOGIC

always @(*)
begin case(state)
  IDLE:
        begin
            if(emergency)
                next_state = EMERGENCY;
            else if(overload)
                next_state = IDLE;
            else if(request_pending &&
                    target_floor > current_floor)
                next_state = MOVE_UP;
            else if(request_pending &&
                    target_floor < current_floor)
                next_state = MOVE_DOWN;
            else if(request_pending &&
                    target_floor == current_floor)
                next_state = DOOR_OPEN;
            else
                next_state = IDLE;
        end

  MOVE_UP:
        begin
            if(emergency)
                next_state = EMERGENCY;
            else if(current_floor == target_floor)
                next_state = DOOR_OPEN;
            else
                next_state = MOVE_UP;
        end

  MOVE_DOWN:
        begin
            if(emergency)
                next_state = EMERGENCY;
            else if(current_floor == target_floor)
                next_state = DOOR_OPEN;
            else
                next_state = MOVE_DOWN;
        end

  DOOR_OPEN:
            next_state = DOOR_CLOSE;

  DOOR_CLOSE:
            next_state = IDLE;

  EMERGENCY:
        begin
            if(emergency)
                next_state = EMERGENCY;
            else
                next_state = IDLE;
        end
        default:
            next_state = IDLE;
      endcase
   end
// OUTPUT + FLOOR CONTROL
always @(posedge clk or posedge reset)
begin
 if(reset)
    begin
        current_floor <= 2'd1;
        door_open <= 1'b0;
        moving_up <= 1'b0;
        moving_down <= 1'b0;
        busy <= 1'b0;
        alarm <= 1'b0;
    end
    else
    begin
        door_open <= 1'b0;
        moving_up <= 1'b0;
        moving_down <= 1'b0;
        busy <= 1'b0;
        alarm <= 1'b0;
        case(state)
  IDLE:
        begin
        busy <= 1'b0;
        if(overload)
                begin
                    alarm <= 1'b1;
                    door_open <= 1'b1;
                end
            end

   MOVE_UP:
            begin
                busy <= 1'b1;
                moving_up <= 1'b1;
          if(slow_tick)
                begin
                if(current_floor < target_floor)
                current_floor <= current_floor + 1'b1;
                end
            end
    MOVE_DOWN:
            begin
                busy <= 1'b1;
                moving_down <= 1'b1;
                if(slow_tick)
                begin
                    if(current_floor > target_floor)
                        current_floor <= current_floor - 1'b1;
                end
            end
     DOOR_OPEN:
            begin
                busy <= 1'b1;
                door_open <= 1'b1;
            end
     DOOR_CLOSE:
            begin
                busy <= 1'b0;
                door_open <= 1'b0;
            end
     EMERGENCY:
            begin
                alarm <= 1'b1;
                door_open <= 1'b1;
                busy <= 1'b0;
                moving_up <= 1'b0;
                moving_down <= 1'b0;
            end
      endcase
    end
end
endmodule
