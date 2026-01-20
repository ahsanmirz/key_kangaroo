#---------------------------------------------------------------------
# Makefile for BSGS
#
# Author : Jean-Luc PONS

ifdef gpu

SRC = SECPK1/IntGroup.cpp main.cpp SECPK1/Random.cpp \
      Timer.cpp SECPK1/Int.cpp SECPK1/IntMod.cpp \
      SECPK1/Point.cpp SECPK1/SECP256K1.cpp \
      GPU/GPUEngine.o Kangaroo.cpp HashTable.cpp \
      Backup.cpp Thread.cpp Check.cpp Network.cpp

OBJDIR = obj

OBJET = $(addprefix $(OBJDIR)/, \
      SECPK1/IntGroup.o main.o SECPK1/Random.o \
      Timer.o SECPK1/Int.o SECPK1/IntMod.o \
      SECPK1/Point.o SECPK1/SECP256K1.o \
      GPU/GPUEngine.o Kangaroo.o HashTable.o Thread.o \
      Backup.o Check.o Network.o)

else

SRC = SECPK1/IntGroup.cpp main.cpp SECPK1/Random.cpp \
      Timer.cpp SECPK1/Int.cpp SECPK1/IntMod.cpp \
      SECPK1/Point.cpp SECPK1/SECP256K1.cpp \
      Kangaroo.cpp HashTable.cpp Thread.cpp Check.cpp \
      Backup.cpp Network.cpp

OBJDIR = obj

OBJET = $(addprefix $(OBJDIR)/, \
      SECPK1/IntGroup.o main.o SECPK1/Random.o \
      Timer.o SECPK1/Int.o SECPK1/IntMod.o \
      SECPK1/Point.o SECPK1/SECP256K1.o \
      Kangaroo.o HashTable.o Thread.o Check.o Backup.o \
      Network.o)

endif

CXX        = g++
# CUDA settings (override from command line if needed, e.g. `make gpu=1 CUDA=/usr/local/cuda ...`)
CUDA       ?= /usr/local/cuda
# CUDA include/lib dirs (handles conda CUDA layouts like /opt/conda/{include,lib})
CUDA_INC   ?= $(CUDA)/include
CUDA_LIB64 := $(wildcard $(CUDA)/lib64)
CUDA_LIB   := $(wildcard $(CUDA)/lib)
CUDA_LIBDIR ?= $(if $(CUDA_LIB64),$(CUDA)/lib64,$(CUDA)/lib)
# Host compiler used by nvcc (override if you need a specific gcc version)
CXXCUDA    ?= $(CXX)
NVCC       ?= $(CUDA)/bin/nvcc

UNAME_S    := $(shell uname -s)
UNAME_M    := $(shell uname -m)

# x86_64-only flags (SSE / -m64). On Apple Silicon (arm64) clang rejects these.
ARCHFLAGS  :=
ifneq ($(filter x86_64,$(UNAME_M)),)
ARCHFLAGS  += -m64 -mssse3
endif

ifdef gpu

ifdef debug
CXXFLAGS   = -DWITHGPU $(ARCHFLAGS) -Wno-unused-result -Wno-write-strings -g -I. -I$(CUDA_INC)
else
CXXFLAGS   = -DWITHGPU $(ARCHFLAGS) -Wno-unused-result -Wno-write-strings -O2 -I. -I$(CUDA_INC)
endif
LFLAGS     = -lpthread -L$(CUDA_LIBDIR) -lcudart

else

ifdef debug
CXXFLAGS   = $(ARCHFLAGS) -Wno-unused-result -Wno-write-strings -g -I.
else
CXXFLAGS   = $(ARCHFLAGS) -Wno-unused-result -Wno-write-strings -O2 -I.
endif
LFLAGS     = -lpthread

endif

#--------------------------------------------------------------------

ifdef gpu
ifdef debug
$(OBJDIR)/GPU/GPUEngine.o: GPU/GPUEngine.cu
	$(NVCC) -G -maxrregcount=0 --ptxas-options=-v --compile --compiler-options -fPIC -ccbin $(CXXCUDA) -m64 -g -I$(CUDA_INC) -gencode=arch=compute_$(ccap),code=sm_$(ccap) -o $(OBJDIR)/GPU/GPUEngine.o -c GPU/GPUEngine.cu
else
$(OBJDIR)/GPU/GPUEngine.o: GPU/GPUEngine.cu
	$(NVCC) -maxrregcount=0 --ptxas-options=-v --compile --compiler-options -fPIC -ccbin $(CXXCUDA) -m64 -O2 -I$(CUDA_INC) -gencode=arch=compute_$(ccap),code=sm_$(ccap) -o $(OBJDIR)/GPU/GPUEngine.o -c GPU/GPUEngine.cu
endif
endif

$(OBJDIR)/%.o : %.cpp
	$(CXX) $(CXXFLAGS) -o $@ -c $<

all: bsgs

bsgs: $(OBJET)
	@echo Making Kangaroo...
	$(CXX) $(OBJET) $(LFLAGS) -o kangaroo

$(OBJET): | $(OBJDIR) $(OBJDIR)/SECPK1 $(OBJDIR)/GPU

$(OBJDIR):
	mkdir -p $(OBJDIR)

$(OBJDIR)/GPU: $(OBJDIR)
	cd $(OBJDIR) && mkdir -p GPU

$(OBJDIR)/SECPK1: $(OBJDIR)
	cd $(OBJDIR) &&	mkdir -p SECPK1

clean:
	@echo Cleaning...
	@rm -f obj/*.o
	@rm -f obj/GPU/*.o
	@rm -f obj/SECPK1/*.o


