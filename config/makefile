# Copyright (c) 2010, Lawrence Livermore National Security, LLC. Produced at the
# Lawrence Livermore National Laboratory. LLNL-CODE-443211. All Rights reserved.
# See file COPYRIGHT for details.
#
# This file is part of the MFEM library. For more information and source code
# availability see http://mfem.org.
#
# MFEM is free software; you can redistribute it and/or modify it under the
# terms of the GNU Lesser General Public License (as published by the Free
# Software Foundation) version 2.1 dated February 1999.

SRC = $(if $(MFEM_DIR),$(MFEM_DIR)/config/,)

# Helper print-info function
mfem-info = $(if $(filter YES,$(VERBOSE)),$(info *** [info]$(1)),)

# The list of all defines that may be enabled in $(CONFIG_HPP) and $(CONFIG_MK)
# is kept in the MFEM_DEFINES variable, exported here from ../makefile

# The list of variables to set in config.mk using their current values
# is kept in the MFEM_CONFIG_VARS variable, exported here from ../makefile

# Debug output
# $(foreach def,$(MFEM_DEFINES),$(info $(def) = $(value $(def))))
# $(foreach var,$(MFEM_CONFIG_VARS),$(info $(var) = $(value $(var))))

# Output path + file name for the config header
CONFIG_HPP = _config.hpp

# Output path + file name for the config makefile
CONFIG_MK = config.mk

.SUFFIXES:
.PHONY: all get-hypre-version header config-mk

all: header config-mk

MPI = $(MFEM_USE_MPI:NO=)
GHV = get_hypre_version
GHV_FLAGS = $(subst @MFEM_DIR@,$(if $(MFEM_DIR),$(MFEM_DIR),..),$(HYPRE_OPT))

$(GHV): $(SRC)$(GHV).cpp
	$(call mfem-info, Determining HYPRE version ...)
	$(MFEM_CXX) ${GHV_FLAGS} $(SRC)$(GHV).cpp -o $(GHV)
$(GHV).out: $(GHV)
	./$(GHV) > $(GHV).out
.INTERMEDIATE: $(GHV) $(GHV).out

get-hypre-version: $(GHV).out
	$(eval MFEM_HYPRE_VERSION:=$(shell cat $(GHV).out))
	$(if $(MFEM_HYPRE_VERSION),$(eval export MFEM_HYPRE_VERSION)\
	$(info HYPRE version: $(MFEM_HYPRE_VERSION)),\
	$(error Unable to determine HYPRE version))

header: $(if $(MPI),get-hypre-version,)
	$(call mfem-info, Writing $(CONFIG_HPP) ...)
	@set -- && \
	for def in $${MFEM_DEFINES} $(if $(MPI),MFEM_HYPRE_VERSION,); do \
	   eval var=\$$$$def && \
	   if [ "NO" != "$${var}" ]; then \
	      set -- "$$@" -e "s|// \(#define $${def} \)|\1|" && \
	      set -- "$$@" -e "s|// \(#define $${def}\)$$|\1|" && \
	      set -- "$$@" -e "s#@$${def}@#$${var}#g"; \
	   fi; \
	done && \
	sed "$$@" $(SRC)config.hpp.in > $(CONFIG_HPP)

# The target below uses the values of the shell variables listed in
# the shell variables MFEM_DEFINES and MFEM_CONFIG_VARS
config-mk:
	$(call mfem-info, Writing $(CONFIG_MK) ...)
	@set -- && \
	for ref in $${MFEM_DEFINES} $${MFEM_CONFIG_VARS}; do \
	   eval var=\$$$$ref && \
	   set -- "$$@" -e "s#@$${ref}@#$${var}#g"; \
	done && \
	set -- "$$@" -e 's/@\([^@]*\)@/$$(\1)/g' -e 's/  *$$//g' && \
	set -- "$$@" -e tb -e ba -e ":b" -e 's/^$$//' -e tc -e ba && \
	set -- "$$@" -e ":c" -e d -e ":a" && \
	sed "$$@" $(SRC)config.mk.in > $(CONFIG_MK)

clean:
	rm -f $(CONFIG_HPP) $(CONFIG_MK) sample-runs-build.log
