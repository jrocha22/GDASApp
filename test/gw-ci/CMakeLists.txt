# Function that generates the 1/2 cycle forecast and DA tasks
function(add_cycling_tests pslot YAML_PATH HOMEgfs RUNTESTS PROJECT_SOURCE_DIR TASK_LIST)
  # Prepare the COMROOT and EXPDIR for the cycling ctests
  add_test(NAME ${pslot}
    COMMAND /bin/bash -c "${PROJECT_SOURCE_DIR}/test/gw-ci/create_exp.sh ${YAML_PATH} ${pslot} ${HOMEgfs} ${RUNTESTS}"
    WORKING_DIRECTORY ${RUNTESTS})
  set_tests_properties(${pslot} PROPERTIES LABELS "manual")

  # Get the 1/2 cycle and full cycle's dates
  execute_process(
    COMMAND ${CMAKE_COMMAND} -E env python ${PROJECT_SOURCE_DIR}/test/gw-ci/get_cycles.py ${YAML_PATH}
    OUTPUT_VARIABLE SCRIPT_OUTPUT
    RESULT_VARIABLE SCRIPT_RESULT
    OUTPUT_STRIP_TRAILING_WHITESPACE
  )
  string(REPLACE "," ";" DATES_LIST ${SCRIPT_OUTPUT})
  list(GET DATES_LIST 0 HALF_CYCLE)
  list(GET DATES_LIST 1 FULL_CYCLE)

  # 1/2 cycle gdasfcst
  message(STATUS "preparing 1/2 cycle gdasfcst for ${pslot} ctest")
  add_test(NAME ${pslot}_gdasfcst_${HALF_CYCLE}
           COMMAND /bin/bash -c "${PROJECT_SOURCE_DIR}/test/gw-ci/run_exp.sh ${pslot} gdasfcst ${HALF_CYCLE}"
           WORKING_DIRECTORY ${RUNTESTS})
  set_tests_properties(${pslot}_gdasfcst_${HALF_CYCLE} PROPERTIES LABELS "manual")

  # Select the list of tasks to run for the full cycle
  message(STATUS "Tasks ${TASK_LIST}")

  foreach(task ${TASK_LIST})
    message(STATUS "preparing the full cycle ${task} for ${pslot} ctest")
    add_test(NAME ${pslot}_${task}_${FULL_CYCLE}
             COMMAND /bin/bash -c "${PROJECT_SOURCE_DIR}/test/gw-ci/run_exp.sh ${pslot} ${task} ${FULL_CYCLE}"
             WORKING_DIRECTORY ${RUNTESTS})
    set_tests_properties(${pslot}_${task}_${FULL_CYCLE} PROPERTIES LABELS "manual")
  endforeach()
endfunction()

# Setup the environement
set(HOMEgfs ${CMAKE_SOURCE_DIR}/../../..)
set(RUNTESTS ${CMAKE_CURRENT_BINARY_DIR}/../../test/gw-ci)

# WCDA, low-res
# -------------
set(pslot "WCDA-3DVAR-C48mx500")
set(YAML_PATH ${HOMEgfs}/ci/cases/pr/C48mx500_3DVarAOWCDA.yaml)
set(TASK_LIST
  "gdasprepoceanobs"
  "gdasocnanalprep"
  "gdasocnanalbmat"
  "gdasocnanalrun"
  "gdasocnanalchkpt"
  "gdasocnanalpost"
  )
add_cycling_tests(${pslot} ${YAML_PATH} ${HOMEgfs} ${RUNTESTS} ${PROJECT_SOURCE_DIR} "${TASK_LIST}")

# Aero-Land DA, C96
# -----------------
set(pslot "Aero-Snow-3DVAR-C96")
set(YAML_PATH ${HOMEgfs}/ci/cases/pr/C96_atmaerosnowDA.yaml)
set(TASK_LIST)  # empty list for now
add_cycling_tests(${pslot} ${YAML_PATH} ${HOMEgfs} ${RUNTESTS} ${PROJECT_SOURCE_DIR} "${TASK_LIST}")

# Atm DA, C96/C48
# ---------------
set(pslot "Atm-hyb-C96C48")
set(YAML_PATH ${HOMEgfs}/ci/cases/pr/C96C48_ufs_hybatmDA.yaml)
set(TASK_LIST)  # empty list for now
add_cycling_tests(${pslot} ${YAML_PATH} ${HOMEgfs} ${RUNTESTS} ${PROJECT_SOURCE_DIR} "${TASK_LIST}")

# GFSv17, 3DVAR prototype
# -----------------------
set(pslot "GFSv17-3DVAR-C384mx025")
set(YAML_PATH ${HOMEgfs}/ci/cases/gfsv17/C384mx025_3DVarAOWCDA.yaml)
set(TASK_LIST
  "gdasprepoceanobs"
  "gdasocnanalprep"
  "gdasocnanalbmat"
  "gdasocnanalrun"
  "gdasocnanalchkpt"
  "gdasocnanalpost"
  "gdasocnanalvrfy"
  "gdasprep"
  "gdasanal"
  )
add_cycling_tests(${pslot} ${YAML_PATH} ${HOMEgfs} ${RUNTESTS} ${PROJECT_SOURCE_DIR} "${TASK_LIST}")
