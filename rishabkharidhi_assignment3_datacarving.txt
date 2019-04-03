#author : Rishab Kharidhi rishab.kharidhi@colorado.edu
#name   : Project 3: Data Carving
#purpose: To perform data carving and extract files from a given file system image 
#date   : 03.12.2019
#version: 3.7

import sys
import os
import hashlib
import binascii

# declaration of global variables
count = 0
stack=[]
filecntr=0
cwdpath=''
pathdir=''
filepath=''

#definition of function to read the file and store it in variable for the code to use
def readfile():
        global data
        fname_obj = open(fname, 'rb')
        data = fname_obj.read()
        fname_obj.close()
        return

#definition of function to create the folder titled as my last name
def createfolder():
        global cwdpath
        global pathdir

        cwdpath = os.getcwd()
        #print("The current working directory is %s" % cwdpath)

        pathdir = cwdpath + "/kharidhi"

        if os.path.isdir(pathdir):
                return
        else:
                os.makedirs(pathdir)
        return

#definition of function to carve the file using sof, eof and file type as arguments
def carvefile(sof, eof, ftype):
        global filecntr
        global fname
        global pathdir
        global filepath

        #print (path)
        filecntr+=1
        filename='file'+str(filecntr)

        if ftype == 'jpg':
                filename='jpg'+filename
                filename+='.jpg'
        if ftype == 'png':
                filename='png'+filename
                filename+='.png'
        if ftype == 'pdf':
                filename='pdf'+filename
                filename+='.pdf'
        filepath=pathdir+"/"+filename

        #print(filepath)
        
        fname_obj = open(fname, 'rb')
        data = fname_obj.read()
        fname_obj.close()
        subdata = data[sof:eof]

        size = eof - sof
        fsize = size/(1024*1024)
        filesize = str(round(fsize, 4))
        
        
        fcarve = open(filepath, 'wb')
        fcarve.write(subdata)
        fcarve.close()
        print ("A " + ftype + " type file has been found at starting offset " + str(hex(sof)) + " and ending offset " + str(hex(eof)) + " having size of " + filesize + " MB has been carved as " +filename)
        hashfile(filepath)
        return

#definition of function to compute the hash of the file that was carved by carvefile()
def hashfile(hashpath):
        global pathdir
        hashfile=pathdir+"/hashes.txt"

        filehash=hashlib.md5(hashpath.encode()).hexdigest()
        
        if os.path.isfile(hashfile):
                fhash = open(hashfile, "a+")
                fhash.write(filehash + "\n")
                fhash.close()

        else:
                fhash = open(hashfile, "w")
                fhash.write(filehash + "\n")
                fhash.close()

        return
                
        

#definition of function to push a value onto a stack        
def push(stack,ptr):
        stack.append(ptr)
        l=len(stack)
        if l ==1:
                return
        for i in range(l-1):
                index=l-i-1
                stack[index]=stack[index-1]
        stack[0]=ptr

#definition of function to find the sof and eof of the jpg files, match the sof to the eof and pass it to the carvefile() function
def findjpg(fname):
        ftype='jpg'
        index = 0
        index2 = 0
        counter = 0
        global filecntr
        global stack

        #opens the file as a read-byte format
        with open(fname, "rb") as f:
                #declaration of states
                state = 0
                state2 = 0
                byte = f.read(1)
                counter = 0
                x=0
                y=0

                #start of the finite state machine loop
                while byte:
                        #part of the loop to match the starting memory offset file signature in the file
                        if state == 0:
                                if byte == b'\xff':
                                        index = counter
                                        state = 1
                                        #print("got ff")
                                else:
                                        state = 0
                        elif state == 1:
                                if byte == b'\xd8':
                                        state = 2
                                        #print("got d8 at", hex(index))
                                elif byte == b'\xff':
                                        index = counter
                                        state = 1
                                else:
                                        state = 0
                        elif state == 2:
                                if byte == b'\xff':
                                        state = 1
                                        #print("detected start at", hex(index))
                                        push(stack,index)
                                        x+=1
                                        index = counter
                                else: 
                                        state = 0
                        else:
                                state = 0


                        #part of the loop to match the ending memory offset file signature in the file
                        if state2 == 0:
                                if byte == b'\xff':
                                        index2=counter
                                        state2 = 1
                                else:
                                        state2 = 0
                        elif state2 == 1:
                                if byte == b'\xd9':
                                        state2 = 0
                                        eof=index2+2
                                        #print("detected end at", hex(eof))
                                        #print(eof)
                                        if len(stack) != 0:
                                                sof=stack.pop()
                                                if sof<index2:
                                                        carvefile(sof, eof, ftype)
                                                if sof>index2:
                                                        push(stack,sof)
                                        y+=1
                                elif byte == b'\xff':
                                        index2 = counter
                                        state2 = 1
                                else:
                                        state2 = 0
                                        
                        byte = f.read(1)
                        counter += 1
                        #print(counter)
                #print(x)
                #print(y)
                filecntr=0
                stack=[]


#definition of function to find the sof and eof of the png files, match the sof to the eof and pass it to the carvefile() function
def findpng(fname):
        ftype='png'
        index = 0
        index2 = 0
        counter = 0
        global filecntr
        global stack
        
        with open(fname, "rb") as f:
                state = 0
                state2 = 0
                byte = f.read(1)
                counter = 0
                x=0
                y=0

                #start of the finite state machine loop
                while byte:

                        #part of the loop to match the starting memory offset file signature in the file
                        if state == 0:
                                if byte == b'\x89':
                                        index = counter
                                        state = 1
                                        #print("got 89")
                                else:
                                        state = 0
                        elif state == 1:
                                if byte == b'\x50':
                                        state = 2
                                        #print("got 50")
                                elif byte == b'\x89':
                                        index = counter
                                        state = 1
                                else:
                                        state = 0
                        elif state == 2:
                                if byte == b'\x4e':
                                        state = 3
                                        #print("got 4e")
                                elif byte == b'\x89':
                                        index = counter
                                        state = 1
                                else:
                                        state = 0
                        elif state == 3:
                                if byte == b'\x47':
                                        state = 4
                                        #print("got 47")
                                elif byte == b'\x89':
                                        index = counter
                                        state = 1
                                else:
                                        state = 0
                        elif state == 4:
                                if byte == b'\x0d':
                                        state = 5
                                        #print("got 0d")
                                elif byte == b'\x89':
                                        index = counter
                                        state = 1
                                else:
                                        state = 0
                        elif state == 5:
                                if byte == b'\x0a':
                                        state = 6
                                        #print("got 0a")
                                elif byte == b'\x89':
                                        index = counter
                                        state = 1
                                else:
                                        state = 0
                        elif state == 6:
                                if byte == b'\x1a':
                                        state = 7
                                        #print("got 1a")
                                elif byte == b'\x89':
                                        index = counter
                                        state = 1
                                else:
                                        state = 0
                        elif state == 7:
                                if byte == b'\x0a':
                                        state = 0
                                        #print("detected start at", hex(index))
                                        push(stack,index)
                                        x+=1
                                elif byte == b'\x89':
                                        index = counter
                                        state = 1
                                else:
                                        state = 0
                        else:
                                state = 0


                        #part of the loop to match the ending memory offset file signature in the file
                        if state2 == 0:
                                if byte == b'\x49':
                                        index2 = counter
                                        state2 = 1
                                        #print("got 49")
                                else:
                                        state2 = 0
                        elif state2 == 1:
                                if byte == b'\x45':
                                        state2 = 2
                                        #print("got 45")
                                elif byte == b'\x49':
                                        index2 = counter
                                        state2 = 1
                                else:
                                        state2 = 0
                        elif state2 == 2:
                                if byte == b'\x4e':
                                        state2 = 3
                                        #print("got 4e")
                                elif byte == b'\x49':
                                        index2 = counter
                                        state2 = 1
                                else:
                                        state2 = 0
                        elif state2 == 3:
                                if byte == b'\x44':
                                        state2 = 4
                                        #print("got 44")
                                elif byte == b'\x49':
                                        index2 = counter
                                        state2 = 1
                                else:
                                        state2 = 0
                        elif state2 == 4:
                                if byte == b'\xae':
                                        state2 = 5
                                        #print("got ae")
                                elif byte == b'\x49':
                                        index2 = counter
                                        state2 = 1
                                else:
                                        state2 = 0
                        elif state2 == 5:
                                if byte == b'\x42':
                                        state2 = 6
                                        #print("got 42")
                                elif byte == b'\x49':
                                        index2 = counter
                                        state2 = 1
                                else:
                                        state2 = 0
                        elif state2 == 6:
                                if byte == b'\x60':
                                        state2 = 7
                                        #print("got 60")
                                elif byte == b'\x49':
                                        index2 = counter
                                        state2 = 1
                                else:
                                        state2 = 0
                        elif state2 == 7:
                                if byte == b'\x82':
                                        state2 = 0
                                        eof = index2+8
                                        #print("detected end at", hex(eof))
                                        if len(stack) != 0:
                                                sof=stack.pop()
                                                if sof<index2:
                                                        carvefile(sof, eof, ftype)
                                                if sof>index2:
                                                        push(stack, sof)
                                        y+=1
                                elif byte == b'\x49':
                                        index2 = counter
                                        state2 = 1
                                else:
                                        state2 = 0
                        byte = f.read(1)
                        counter += 1
                #print(x)
                #print(y)
                filecntr=0
                stack=[]

#definition of function to find the sof and eof of the pdf files, match the sof to the eof and pass it to the carvefile() function
def findpdf(fname):
        ftype='pdf'
        index = 0
        index2 = 0
        counter = 0
        global filecntr
        global stack
        
        with open(fname, "rb") as f:
                state = 0
                state2 = 0
                byte = f.read(1)
                counter = 0
                x=0
                y=0

                #start of the finite state machine loop
                while byte:

                        #part of the loop to match the starting memory offset file signature in the file
                        if state == 0:
                                if byte == b'\x25':
                                        index = counter
                                        state = 1
                                        #print("got 25")
                                else:
                                        state = 0
                        elif state == 1:
                                if byte == b'\x50':
                                        state = 2
                                        #print("got 50")
                                elif byte == b'\x25':
                                        index = counter
                                        state = 1
                                else:
                                        state = 0
                        elif state == 2:
                                if byte == b'\x44':
                                        state = 3
                                        #print("got 44")
                                elif byte == b'\x25':
                                        index = counter
                                        state = 1
                                else:
                                        state = 0
                        elif state == 3:
                                if byte == b'\x46':
                                        state = 0
                                        #print("detected start at", hex(index))
                                        push(stack,index)
                                        x+=1
                                elif byte == b'\x25':
                                        index = counter
                                        state = 1
                                else:
                                        state = 0
                        else:
                                state = 0



                        #part of the loop to match the ending memory offset file signature in the file
                        if state2 == 0:
                                if byte == b'\x0a':
                                        index2 = counter
                                        state2 = 1
                                        #print("got 0a")
                                else:
                                        state2 = 0
                        elif state2 == 1:
                                if byte == b'\x25':
                                        state2 = 2
                                        #print("got 25")
                                elif byte == b'\x0a':
                                        index2 = counter
                                        state2 = 1
                                else:
                                        state2 = 0
                        elif state2 == 2:
                                if byte == b'\x25':
                                        state2 = 3
                                        #print("got 25")
                                elif byte == b'\x0a':
                                        index2 = counter
                                        state2 = 1
                                else:
                                        state2 = 0
                        elif state2 == 3:
                                if byte == b'\x45':
                                        state2 = 4
                                        #print("got 45")
                                elif byte == b'\x0a':
                                        index2 = counter
                                        state2 = 1
                                else:
                                        state2 = 0
                        elif state2 == 4:
                                if byte == b'\x4f':
                                        state2 = 5
                                        #print("got 4f")
                                elif byte == b'\x0a':
                                        index2 = counter
                                        state2 = 1
                                else:
                                        state2 = 0
                        elif state2 == 5:
                                if byte == b'\x46':
                                        state2 = 0
                                        #print("got 46")
                                        eof = index2+10
                                        #print("detected end at", hex(eof))
                                        if len(stack) != 0:
                                                for sof in stack:
                                                        if sof<index2:
                                                                carvefile(sof, eof, ftype)
                                        y+=1
                                elif byte == b'\x0a':
                                        index2 = counter
                                        state2 = 1
                                else:
                                        state2 = 0
                        byte = f.read(1)
                        counter += 1
                #print(x)
                #print(y)
                filecntr=0
                stack=[]
                


#main part of the code
if __name__ == "__main__":
        
        fname = input("Enter a file name: ")
        createfolder()

        if os.path.isfile(fname):
                findjpg(fname)
                findpng(fname)
                findpdf(fname)
        else:
                print("Given file does not exist!")

