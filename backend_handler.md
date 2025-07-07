# LLVM源码阅读(LLVM-21)--后端处理概述

> 作者：王鑫  
> 日期：2025-07-07  
> 标签：LLVM、编译器、DAG、优化

---

## 📝 摘要

简要介绍文章内容：

---

## 📌 目录

- [摘要](#-摘要)
- [前言](#前言)
- [正文](#正文)
  - [函数调用链汇总](###1.函数调用链汇总)
- [参考资料](#参考资料)

---

## 前言

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

## 正文
### 1. 函数调用链汇总

>//入口
> SelectionDAGISelPass::run()
> ->SelectionDAGISel::runOnMachineFunction()
> -->SelectionDAGISel::SelectAllBasicBlocks()
> --->SelectionDAGISel::CodeGenAndEmitDAG()
> //DAG优化（合并/转换）
> ---->SelectionDAG::Combine()
> //通用的combine
> ----->DAGCombiner::visit()
> ------>DAGCombiner::visitABS()
> ------->DAGCombiner::foldABSToABD()
> -------->SelectionDAG::getNode()
> //目标机专有的combine
> ----->TargetLowering::PerformDAGCombine
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
> ------>SelectionDAG::getNode()

### 3. 分模块介绍
构建DAG结点->DAG结点优化->指令模式选择->生成目标汇编
#### 3.1 构建DAG结点

> SelectionDAGISel::runOnMachineFunction()
> ->SelectionDAGISel::SelectAllBasicBlocks()
> -->For each basic block:
> --->SelectionDAGBuilder::visit()
> ---->SelectionDAGBuilder::visitAdd()
> ----->SelectionDAGBuilder::visitBinary()
> ------>SelectionDAG::getNode()

#### 3.1 DAG结点优化
> --->SelectionDAGISel::CodeGenAndEmitDAG()
> //DAG优化（合并/转换）
> ---->SelectionDAG::Combine()
> //通用的combine
> ----->DAGCombiner::visit()
> ------>DAGCombiner::visitABS()
> ------->DAGCombiner::foldABSToABD()
> -------->SelectionDAG::getNode()
> //目标机专有的combine
> ----->TargetLowering::PerformDAGCombine

#### 3.1 指令模式选择

> --->SelectionDAGISel::CodeGenAndEmitDAG()
> //指令选择。
> ---->SelectionDAGISel::DoInstructionSelection()
> //或者使用手动选择
> ----->RISCVDAGToDAGISel::Select()
> // 匹配.td生成的模式
> ----->RISCVDAGToDAGISel::SelectCode()

#### 3.1 生成目标汇编

> //汇编输出
> ----->InstrEmitter::EmitNode()
> //机器指令放入基本块中
> ------>InstrEmitter::EmitMachineNode()
//推荐
> ------->MachineInstrBuilder::BuildMI()
//更底层，不推荐
> ------->MachineBasicBlock::instert()


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTgyNjg2MTUzNF19
-->