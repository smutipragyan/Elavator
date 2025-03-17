# Elavator
# verilog code for designing
module Elevator_Controller(
    input clk, reset,
    input [2:0] floor_request,  // Buttons for 0, 1, 2 floors
    input overload,             // Overload sensor
    input door_obstruction,     // Safety mechanism for doors
    output reg [1:0] current_floor, // 00 = Floor 0, 01 = Floor 1, 10 = Floor 2
    output reg door_open,       // Door state
    output reg moving_up, moving_down // Elevator movement indicators
);

    // State Encoding
    typedef enum reg [2:0] {IDLE, MOVE_UP, MOVE_DOWN, OPEN_DOOR} state_t;
    state_t state, next_state;

    always @(posedge clk or posedge reset) begin
        if (reset)
            state <= IDLE;
        else
            state <= next_state;
    end

    always @(*) begin
        // Default outputs
        moving_up = 0; moving_down = 0; door_open = 0;
        next_state = state;

        case (state)
            IDLE: begin
                if (current_floor <= 2 && floor_request[current_floor] && !overload) 
                    next_state = OPEN_DOOR;
                else if (floor_request > {1'b0, current_floor}) // Convert 2-bit to 3-bit
                    next_state = MOVE_UP;
                else if (floor_request < {1'b0, current_floor}) 
                    next_state = MOVE_DOWN;
            end

            MOVE_UP: begin
                moving_up = 1;
                if (current_floor < 2 && floor_request[current_floor + 1])
                    next_state = OPEN_DOOR;
                else
                    next_state = MOVE_UP;
            end

            MOVE_DOWN: begin
                moving_down = 1;
                if (current_floor > 0 && floor_request[current_floor - 1])
                    next_state = OPEN_DOOR;
                else
                    next_state = MOVE_DOWN;
            end

            OPEN_DOOR: begin
                door_open = 1;
                if (!door_obstruction && !overload) // Doors close only if safe
                    next_state = IDLE;
            end

        endcase
    end

    always @(posedge clk) begin
        if (state == MOVE_UP && current_floor < 2)
            current_floor <= current_floor + 1;
        else if (state == MOVE_DOWN && current_floor > 0)
            current_floor <= current_floor - 1;
    end

endmodule  
