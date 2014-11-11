PROJECT ( klee )
CMAKE_MINIMUM_REQUIRED ( VERSION 2.8.8 FATAL_ERROR )


#########################
# Configure Klee
#########################

SET ( CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake/modules" )

FIND_PACKAGE ( LLVM 3.4 REQUIRED all-targets bitwriter linker ipo instrumentation support tablegen asmparser )

# Configuration part
INCLUDE ( CheckIncludeFiles )
INCLUDE ( CheckFunctionExists )

CHECK_INCLUDE_FILES ( malloc.h HAVE_MALLOC_H )
CHECK_INCLUDE_FILES ( stdlib.h HAVE_STDLIB_H )
CHECK_INCLUDE_FILES ( stdint.h HAVE_STDINT_H )
CHECK_INCLUDE_FILES ( string.h HAVE_STRING_H )
CHECK_INCLUDE_FILES ( strings.h HAVE_STRINGS_H )
CHECK_INCLUDE_FILES ( sys/acl.h HAVE_SYS_ACL_H )
CHECK_INCLUDE_FILES ( sys/capability.h HAVE_SYS_CAPABILITY_H )
CHECK_INCLUDE_FILES ( sys/stat.h HAVE_SYS_STAT_H )
CHECK_INCLUDE_FILES ( sys/types.h HAVE_SYS_TYPES_H )
CHECK_INCLUDE_FILES ( unistd.h HAVE_UNISTD_H )
CHECK_INCLUDE_FILES ( selinux/selinux.h HAVE_SELINUX_SELINUX_H )
CHECK_INCLUDE_FILES ( memory.h HAVE_MEMORY_H )

IF ( HAVE_MALLOC_H )
CHECK_FUNCTION_EXISTS ( mallinfo HAVE_MALLINFO )
ENDIF ( HAVE_MALLOC_H )

IF ( NOT  CMAKE_BUILD_TYPE )
    SET ( CMAKE_BUILD_TYPE "Debug" )
ENDIF ( NOT CMAKE_BUILD_TYPE )


##########################
# Add subprojects
##########################

ADD_SUBDIRECTORY ( stp )
INCLUDE_DIRECTORIES ( stp/include ${PROJECT_BINARY_DIR}/stp/include )
SET (LINK_LIBRARIES ${LINK_LIBRARIES} stp )

IF ( EXISTS "${PROJECT_SOURCE_DIR}/metaSMT/CMakeLists.txt" )
    SET ( metaSMT_USE_SMT2       off )
    SET ( metaSMT_USE_SWORD      off )
    SET ( metaSMT_USE_Boolector  off )
    SET ( metaSMT_USE_Z3         on )
    SET ( metaSMT_USE_STP        on )
    SET ( metaSMT_USE_CUDD       off )
    SET ( metaSMT_USE_MiniSat    off )
    SET ( metaSMT_USE_picosat    off )
    SET ( metaSMT_USE_Aiger      off )
    SET ( metaSMT_USE_CW         off )
    SET ( metaSMT_USE_ExprSolver off )
    
    SET ( metaSMT_DOC_Doxygen    off )
    SET ( metaSMT_ENABLE_TESTS   off )
    SET ( metaSMT_ENABLE_BINDINGS on )
    SET ( metaSMT_ENABLE_TOOLBOX  off )

    ADD_SUBDIRECTORY ( metaSMT )
    INCLUDE_DIRECTORIES ( metaSMT/src )
    SET ( LINK_LIBRARIES ${LINK_LIBRARIES} metaSMT )
    SET ( SUPPORT_METASMT 1 )
ENDIF ()


##########################
# Build klee 
##########################

CONFIGURE_FILE(${PROJECT_SOURCE_DIR}/include/klee/Config/config.h.cmake.in ${PROJECT_BINARY_DIR}/include/klee/Config/config.h)

# Library building part



SET ( KLEE_LIB_FILES 
	./lib/Basic/CmdLineOptions.cpp
	./lib/Basic/ConstructSolverChain.cpp
	./lib/Basic/KTest.cpp
	./lib/Basic/Statistics.cpp
	./lib/Core/AddressSpace.cpp
	./lib/Core/CallPathManager.cpp
	./lib/Core/Common.cpp
	./lib/Core/Context.cpp
	./lib/Core/CoreStats.cpp
	./lib/Core/ExecutionState.cpp
	./lib/Core/Executor.cpp
	./lib/Core/ExecutorTimers.cpp
	./lib/Core/ExecutorUtil.cpp
	./lib/Core/ExternalDispatcher.cpp
	./lib/Core/ImpliedValue.cpp
	./lib/Core/Memory.cpp
	./lib/Core/MemoryManager.cpp
	./lib/Core/PTree.cpp
	./lib/Core/Searcher.cpp
	./lib/Core/SeedInfo.cpp
	./lib/Core/SpecialFunctionHandler.cpp
	./lib/Core/StatsTracker.cpp
	./lib/Core/TimingSolver.cpp
	./lib/Core/UserSearcher.cpp
	./lib/Expr/Constraints.cpp
	./lib/Expr/Expr.cpp
	./lib/Expr/ExprBuilder.cpp
	./lib/Expr/ExprEvaluator.cpp
	./lib/Expr/ExprPPrinter.cpp
	./lib/Expr/ExprSMTLIBLetPrinter.cpp
	./lib/Expr/ExprSMTLIBPrinter.cpp
	./lib/Expr/ExprUtil.cpp
	./lib/Expr/ExprVisitor.cpp
	./lib/Expr/Lexer.cpp
	./lib/Expr/Parser.cpp
	./lib/Expr/Updates.cpp
	./lib/Module/Checks.cpp
	./lib/Module/InstructionInfoTable.cpp
	./lib/Module/IntrinsicCleaner.cpp
	./lib/Module/KInstruction.cpp
	./lib/Module/KModule.cpp
	./lib/Module/LowerSwitch.cpp
	./lib/Module/ModuleUtil.cpp
	./lib/Module/Optimize.cpp
	./lib/Module/PhiCleaner.cpp
	./lib/Module/RaiseAsm.cpp
#	./lib/SMT/main.cpp
#	./lib/SMT/SMTParser.cpp
	./lib/Solver/CachingSolver.cpp
	./lib/Solver/CexCachingSolver.cpp
	./lib/Solver/ConstantDivision.cpp
	./lib/Solver/FastCexSolver.cpp
	./lib/Solver/IncompleteSolver.cpp
	./lib/Solver/IndependentSolver.cpp
	./lib/Solver/PCLoggingSolver.cpp
	./lib/Solver/QueryLoggingSolver.cpp
	./lib/Solver/SMTLIBLoggingSolver.cpp
	./lib/Solver/Solver.cpp
	./lib/Solver/SolverStats.cpp
	./lib/Solver/STPBuilder.cpp
#	./lib/Support/MemoryUsage.cpp
	./lib/Support/RNG.cpp
	./lib/Support/Time.cpp
	./lib/Support/Timer.cpp
	./lib/Support/TreeStream.cpp)

SET ( KLEE_TEST_FILES
	./test/CXX/ArrayNew.cpp
	./test/CXX/New.cpp
	./test/CXX/SimpleVirtual.cpp
	./test/CXX/StaticConstructor.cpp
	./test/CXX/StaticDestructor.cpp
	./test/CXX/Trivial.cpp
	./test/Dogfood/ImmutableSet.cpp
	./test/Feature/CallToUndefinedExternal.cpp
	./test/Feature/LargeReturnTypes.cpp
	./test/Feature/LongDouble.cpp
	./tools/gen-random-bout/gen-random-bout.cpp
	./tools/kleaver/main.cpp
	./tools/klee/Debug.cpp
	./tools/klee/main.cpp
	./unittests/Expr/ExprTest.cpp
	./unittests/Ref/RefTest.cpp
	./unittests/Solver/SolverTest.cpp
	./unittests/TestMain.cpp)

SET_SOURCE_FILES_PROPERTIES ( ./lib/Solver/Solver.cpp PROPERTIES COMPILE_FLAGS "-I${PROJECT_SOURCE_DIR}/lib/Solver" )
SET_SOURCE_FILES_PROPERTIES ( ./lib/Solver/STPBuilder.cpp PROPERTIES COMPILE_FLAGS "-I${PROJECT_SOURCE_DIR}/lib/Solver" )

INCLUDE_DIRECTORIES ( 
	"${PROJECT_SOURCE_DIR}/include" 
	"${PROJECT_BINARY_DIR}/include" 
	${LLVM_INCLUDE_DIRS})
ADD_DEFINITIONS ( -D__STDC_LIMIT_MACROS -D__STDC_CONSTANT_MACROS )

SET ( CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} -std=c++11 )

ADD_LIBRARY( klee ${KLEE_LIB_FILES} )
TARGET_LINK_LIBRARIES ( klee ${METASMT_LIBRARIES})
