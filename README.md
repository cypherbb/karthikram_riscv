# karthikram_riscv
DAY3 <br>
Please review the screenshots in the folders <br>
DAY4- Basic RISC-V micro-architecture <br>
Please review the screenshots in the folder
  ```
  
   |cpu
      @0
         $reset = *reset;
         $pc[31:0] = >>1$reset? 32'b0 :
                     >>1$taken_br ? >>1$br_tgt_pc :
                     >>1$pc + 32'b100;
         $imem_rd_en = !$reset;
         $imem_rd_addr[M4_IMEM_INDEX_CNT-1 : 0] = $pc[M4_IMEM_INDEX_CNT+1 : 2] ;
         
      @1

         $instr[31:0] = $imem_rd_data[31:0];
         //opcode_decode
         $is_i_instr = $instr[6:2] ==? 5'b0000x || 
                       $instr[6:2] ==? 5'b001x0 || 
                       $instr[6:2] ==? 5'b11001 || 
                       $instr[6:2] ==? 5'b11100 ;
         $is_s_instr = $instr[6:2] ==? 5'b0100x ;
         $is_r_instr = $instr[6:2] ==? 5'b011x0 || 
                       $instr[6:2] ==? 5'b01011 || 
                       $instr[6:2] ==? 5'b10100;
         $is_j_instr = $instr[6:2] ==? 5'b11011;
         $is_u_instr = $instr[6:2] ==? 5'b0x101;
         $is_b_instr = $instr[6:2] ==? 5'b11000;
         
         //imm
         $imm[31:0] = $is_i_instr ? { {21{$instr[31]}}, $instr[30:20] } :
                      $is_s_instr ? { {21{$instr[31]}}, $instr[30:25], $instr[11:7] }:
                      $is_b_instr ? { {20{$instr[31]}}, $instr[7], $instr[30:25], $instr[11:8], 1'b0 } :
                      $is_u_instr ? { $instr[31:12], 12'b0 } :
                      $is_j_instr ? { {12{$instr[31]}}, $instr[19:12], $instr[20], $instr[30:21], 1'b0 } :
                      32'b0 ;
         //extraction of instruction fields
         $opcode[6:0] = $instr[6:0];
         ?$funct7_valid
            $funct7[6:0] = $instr[31:25];
         ?$funct3_valid
            $funct3[2:0] = $instr[14:12];
         ?$rs1_valid
            $rs1[4:0] = $instr[19:15];
         ?$rs2_valid
            $rs2[4:0] = $instr[24:20];
         ?$rd_valid
            $rd[4:0] = $instr[11:7];
         
         // validity expressions for the above instructions 
         $rs2_valid = $is_r_instr || $is_s_instr || $is_b_instr;
         $funct3_valid = $is_r_instr || $is_s_instr || $is_b_instr || $is_i_instr;
         $rs1_valid = $is_r_instr || $is_s_instr || $is_b_instr || $is_i_instr;
         $rd_valid =  $is_r_instr || $is_u_instr || $is_j_instr || $is_i_instr;
         $funct7_valid = $is_r_instr;
         
         //decoding individual instructions
         $dec_bits[10:0] = 
            {$funct7[5], $funct3, $opcode};
         
         //branch instructions
         $is_beq = $dec_bits ==?
                   11'bx_000_1100011;
         $is_bne = $dec_bits ==?
                   11'bx_001_1100011;
         $is_blt = $dec_bits ==?
                   11'bx_100_1100011;
         $is_bge = $dec_bits ==?
                   11'bx_101_1100011;
         $is_bltu = $dec_bits ==?
                   11'bx_110_1100011;
         $is_bgeu = $dec_bits ==?
                   11'bx_111_1100011;
         
         //addi
         $is_addi = $dec_bits ==?
                   11'bx_000_0010011;
         $is_add = $dec_bits ==?
                   11'b0_000_0010011;
         
         //register file read
         $rf_wr_en = $rd_valid && $rd != 5'b0;
         $rf_wr_index[4:0] = $rd;
         $rf_wr_data[31:0] = $result;
         
         
         $rf_rd_en1 = $rs1_valid;
         $rf_rd_en2 = $rs2_valid;
         
         $rf_rd_index1[4:0] = $rs1;
         $rf_rd_index2[4:0] = $rs2;
         
         $src1_value[31:0] = $rf_rd_data1[31:0];
         $src2_value[31:0] = $rf_rd_data2[31:0];
         
         //IE
         $result[31:0] = 
            $is_addi ? $src1_value + $imm :
            $is_add ? $src1_value + $src2_value :
            32'bx;
         $taken_br = $is_beq ? ($src1_value == $src2_value) :
                     $is_bne ? ($src1_value != $src2_value) :
                     $is_blt ? (($src1_value < $src2_value) ^ ($src1_value[31] != $src2_value[31]))  :
                     $is_bge ? (($src1_value >= $src2_value) ^ ($src1_value[31] != $src2_value[31])) :
                     $is_bltu ? (($src1_value < $src2_value)) :
                     $is_bgeu ? (($src1_value >= $src2_value)) :
                     1'b0;
         
         //branch instructions
         $br_tgt_pc[31:0] = $pc + $imm;
  ```
