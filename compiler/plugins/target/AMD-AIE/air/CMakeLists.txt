# Copyright 2023 The IREE Authors
#
# Licensed under the Apache License v2.0 with LLVM Exceptions.
# See https://llvm.org/LICENSE.txt for license information.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception

set(IREE_PACKAGE_ROOT_DIR "${CMAKE_CURRENT_LIST_DIR}")
set(IREE_PACKAGE_ROOT_PREFIX "iree::target::amd-aie::air")

list(APPEND IREE_COMPILER_TABLEGEN_INCLUDE_DIRS
    "${IREE_MLIR_AIR_SOURCE_DIR}/include")

iree_cc_library(
  NAME
    defs
  INCLUDES
    "${IREE_MLIR_AIR_SOURCE_DIR}/include"
)

###############################################################################
# AIR Dialect
###############################################################################

iree_cc_library(
    NAME
        AIRDialectIR
    SRCS
      ${IREE_MLIR_AIR_SOURCE_DIR}/lib/Dialect/AIR/IR/AIRDialect.cpp
    DEPS
      ::defs
      ::AIRDialectGen
      ::AIRInterfaceGen
      ::AIRTransformOpsGen
      MLIRIR
)

iree_tablegen_library(
  NAME
    AIRDialectGen
  TD_FILE
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Dialect/AIR/AIR.td"
  OUTS
    -gen-op-decls Dialect/AIR/AIR.h.inc
    -gen-op-defs Dialect/AIR/AIR.cpp.inc
    -gen-typedef-decls Dialect/AIR/AIRTypes.h.inc
    -gen-typedef-defs Dialect/AIR/AIRTypes.cpp.inc
    -gen-dialect-decls Dialect/AIR/AIRDialect.h.inc --dialect=air
    -gen-dialect-defs Dialect/AIR/AIRDialect.cpp.inc --dialect=air
    -gen-enum-decls Dialect/AIR/AIREnums.h.inc
    -gen-enum-defs Dialect/AIR/AIREnums.cpp.inc
)

iree_tablegen_library(
  NAME
    AIRInterfaceGen
  TD_FILE
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Dialect/AIR/AIROpBase.td"
  OUTS
    -gen-op-interface-decls Dialect/AIR/AIROpInterfaces.h.inc
    -gen-op-interface-defs Dialect/AIR/AIROpInterfaces.cpp.inc
)

iree_cc_library(
  NAME
    AIRTransformOps
  SRCS
    ${IREE_MLIR_AIR_SOURCE_DIR}/lib/Dialect/AIR/TransformOps/AIRTransformOps.cpp
  DEPS
    ::defs
    ::AIRDialectIR
    ::AIRTransformOpsGen
    ::AIRTransformPasses
    iree::target::amd-aie::aie::AIEDialectIR
    MLIRIR
    MLIRLinalgTransformOps
)

iree_tablegen_library(
  NAME
    AIRTransformOpsGen
  TD_FILE
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Dialect/AIR/AIRTransformOps.td"
  OUTS
    -gen-op-decls Dialect/AIR/AIRTransformOps.h.inc
    -gen-op-defs Dialect/AIR/AIRTransformOps.cpp.inc
)

###############################################################################
# AIRRt Dialect
###############################################################################

iree_cc_library(
    NAME
      AIRRtDialectIR
    SRCS
      ${IREE_MLIR_AIR_SOURCE_DIR}/lib/Dialect/AIRRt/IR/AIRRtDialect.cpp
      ${IREE_MLIR_AIR_SOURCE_DIR}/lib/Dialect/AIRRt/IR/AIRRtOps.cpp
    DEPS
      ::defs
      ::AIRRtDialectGen
      MLIRIR
)

iree_tablegen_library(
  NAME
    AIRRtDialectGen
  TD_FILE
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Dialect/AIRRt/AIRRtOps.td"
  OUTS
    -gen-op-decls Dialect/AIRRt/AIRRtOps.h.inc
    -gen-op-defs Dialect/AIRRt/AIRRtOps.cpp.inc
    -gen-typedef-decls Dialect/AIRRt/AIRRtOpsTypes.h.inc
    -gen-typedef-defs Dialect/AIRRt/AIRRtOpsTypes.cpp.inc
    -gen-dialect-decls Dialect/AIRRt/AIRRtOpsDialect.h.inc --dialect=airrt
    -gen-dialect-defs Dialect/AIRRt/AIRRtOpsDialect.cpp.inc --dialect=airrt
    -gen-enum-decls Dialect/AIRRt/AIRRtOpsEnums.h.inc
    -gen-enum-defs Dialect/AIRRt/AIRRtOpsEnums.cpp.inc
)

###############################################################################
# AIR Conversion Passes
###############################################################################

iree_tablegen_library(
  NAME
    AIRConversionPassesIncGen
  TD_FILE
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Conversion/Passes.td"
  OUTS
    -gen-pass-decls Conversion/Passes.h.inc
)

iree_cc_library(
  NAME
    AIRConversionPassHeaders
  HDRS
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Conversion/PassDetail.h"
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Conversion/Passes.h"
    "Passes.h.inc"
  DEPS
    ::AIRConversionPassesIncGen
    MLIRPass
  PUBLIC
)

iree_cc_library(
  NAME
    AIRUtil
  HDRS
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Util/DirectedAdjacencyMap.h"
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Util/Dependency.h"
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Util/Util.h"
  SRCS
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Util/DirectedAdjacencyMap.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Util/Dependency.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Util/DirectedAdjacencyMap.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Util/Util.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Util/Outliner.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Util/CostModel.cpp"
  DEPS
    ::defs
    ::AIRDialectIR
    MLIRIR
    MLIRTransforms
)

iree_cc_library(
  NAME
    AIRConversionPasses
  HDRS
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Conversion/ConvertToAIRPass.h"
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Conversion/Passes.h"
  SRCS
    "ConversionPasses.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Conversion/ConvertToAIRPass.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Conversion/AIRLoweringPass.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Conversion/AIRRtToNpuPass.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Conversion/AIRToAIEPass.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Conversion/AIRToAIESchedulingUtils.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Conversion/AIRPipeline.cpp"
  DEPS
    ::defs
    ::AIRConversionPassHeaders
    ::AIRTransform
    ::AIRTransformOps
    ::AIRDialectIR
    ::AIRRtDialectIR
    iree::target::amd-aie::aie::AIEDialectIR
    iree::target::amd-aie::aie::AIEXDialectIR
    MLIRIR
    MLIRLinalgTransforms
    MLIRLinalgUtils
    MLIRSupport
    MLIRTransforms
    MLIRPass
)

###############################################################################
# AIR Transform Passes
###############################################################################

iree_tablegen_library(
  NAME
    AIRTransform
  TD_FILE
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Transform/Passes.td"
  OUTS
    -gen-pass-decls Transform/Passes.h.inc
)

iree_tablegen_library(
  NAME
    AIRTransformPassesIncGen
  TD_FILE
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Transform/Passes.td"
  OUTS
    -gen-pass-decls Transform/Passes.h.inc
)

iree_cc_library(
  NAME
    AIRTransformPassHeaders
  HDRS
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Transform/PassDetail.h"
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Transform/Passes.h"
    "Passes.h.inc"
  DEPS
    ::AIRTransformPassesIncGen
    ::AIRRtDialectIR
    MLIRPass
  PUBLIC
)

iree_cc_library(
  NAME
    AIRTransformPasses
  HDRS
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Transform/AIRDependency.h"
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Transform/AIRMiscPasses.h"
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Transform/Passes.h"
  SRCS
    "TransformPasses.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Transform/AIRHerdPlacementPass.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Transform/AIRLinalgCodegen.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Transform/AffineLoopOptPass.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Transform/AIRMiscPasses.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Transform/AIRDependency.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Transform/AIRDependencyScheduleOpt.cpp"
    "${IREE_MLIR_AIR_SOURCE_DIR}/lib/Transform/AIRDependencyCanonicalize.cpp"
  DEPS
    ::defs
    iree::target::amd-aie::aie::AIEDialectIR
    ::AIRDialectIR
    ::AIRTransform
    ::AIRTransformPassHeaders
    ::AIRUtil
    MLIRIR
    MLIRLinalgDialect
    MLIRLinalgTransforms
    MLIRLinalgUtils
    MLIRSupport
)

###############################################################################
# AIR Passes
###############################################################################

iree_cc_library(
  NAME
    AIRPasses
  HDRS
    "${IREE_MLIR_AIR_SOURCE_DIR}/include/air/Transform/Passes.h"
  SRCS
    "Passes.cpp"
  DEPS
    ::AIRConversionPasses
    ::AIRTransformPasses
)
