## LLVM源码阅读(LLVM-21)--后端处理概述

 1. 前言

后端处理的起始点是IR中间语言函数，例如：

> define i32 @add_func(i32 %a, i32 %b) {
entry:
  %sum = add i32 %a, %b
  ret i32 %sum
}
目

标是翻译成机器码，例如：

> addl %esi, %edi
movl %edi, %eax
ret

下面从函数调用链的角度说明这个过程经历的主要阶段

 2. 函数调用链汇总

>//入口
> SelectionDAGISelPass::run()
> ->SelectionDAGISel::runOnMachineFunction()
> -->SelectionDAGISel::SelectAllBasicBlocks()
> --->SelectionDAGISel::CodeGenAndEmitDAG()
> //DAG优化（合并/转换）
> ---->SelectionDAG::Combine()
> //指令选择。
> ---->SelectionDAGISel::DoInstructionSelection()
> //或者使用手动选择
> ----->RISCVDAGToDAGISel::Select()
> // 匹配.td生成的模式
> ----->RISCVDAGToDAGISel::SelectCode()
> ---->ScheduleDAGSDNodes::run()
> //生成机器指令
> ----->ScheduleDAGVLIW::Schedule()
> ---->ScheduleDAGSDNodes::EmitSchedule()
> //汇编输出
> ----->InstrEmitter::EmitNode()
> //机器指令放入基本块中
> ------>InstrEmitter::EmitMachineNode()
//推荐
> ------->MachineInstrBuilder::BuildMI()
//更底层，不推荐
> ------->MachineBasicBlock::instert()
> -->SelectionDAGISel::SelectBasicBlock()
> --->SelectionDAGBuilder::visit()
> ---->SelectionDAGBuilder::visit()
> //(#include "llvm/IR/Instruction.def") visitAdd()
> ----->Instruction::OPCODE: visit##OPCODE
> ----->SelectionDAGBuilder::visitAdd(const User &I) {visitBinary(I, ISD::ADD);}

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI1MjQ2NzY3N119
-->