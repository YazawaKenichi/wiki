PROGRAM = Project

CC = g++
CONT = cpp
LIBS = -lm

MAIN = main
SUB1 = sub1
SUB2 = sub2

SRCS = ./src
INCS = ./inc
BINS = ./bin
BUILD = ./build

CFLAGS = -I$(INCS)

OBJS = $(BINS)/$(MAIN).o $(BINS)/$(SUB1).o $(BINS)/$(SUB2).o
CODES = $(SRCS)/$(MAIN).$(CONT) $(SRCS)/$(SUB1).$(CONT) $(SRCS)/$(SUB2).$(CONT)

# ↓こいつが機能してない
.cpp.o:
	$(CC) $(CFLAGS) -o $@ -c $<

$(PROGRAM): $(OBJS)
	$(CC) $(CFLAGS) -o $(BUILD)/$@ $^

all: clean $(PROGRAM)

clean:
	rm -rf $(BUILD) $(BINS)
	mkdir $(BUILD) $(BINS)

$(BINS)/$(MAIN).o: $(SRCS)/$(MAIN).$(CONT)
	$(CC) $(CFLAGS) -o $@ -c $<
$(BINS)/$(SUB1).o: $(SRCS)/$(SUB1).$(CONT)
	$(CC) $(CFLAGS) -o $@ -c $<
$(BINS)/$(SUB2).o: $(SRCS)/$(SUB2).$(CONT)
	$(CC) $(CFLAGS) -o $@ -c $<

