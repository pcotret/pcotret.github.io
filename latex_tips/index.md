# LateX tips


## Generating random numbers in LateX

The following code will generate a random number within a range for each compilation

```tex
\pgfmathsetseed{\number\pdfrandomseed}
\newcommand{\thecmd}[2]{ 
	\pgfmathsetmacro{\thenum}{int(random(#1,#2))}
	\thenum
}
```

## Using `ttfamily` with `bfseries` in a listing

Default font doesn't implement bold style:
```latex
\renewcommand{\ttdefault}{pcr}
\begin{lstlisting}[basicstyle=\ttfamily\bfseries]
y:=2
\end{lstlisting}
```

## Makefile to compile a LateX project
```bash
## Here is a simple Makefile for a basic LaTeX flow with a bibliography
## make help:
##     print this menu
## make all:
##     compile the stuff
## make clean:
##     remove temporary files
## make clean_pdf:
##     remove the output PDF file
## make clean_all:
##     remove EVERYTHING
# Variables
FILENAME=mainfile
BIBNAME=mainfile

help:
	@grep -e "^##" Makefile;
all:
	pdflatex ${FILENAME}.tex
	bibtex ${FILENAME}
	pdflatex ${FILENAME}.tex
	pdflatex ${FILENAME}.tex
clean:
	rm *.aux *.bbl *.blg *.log *.out *.snm *.toc *.vrb *.xml main-blx.bib *.nav
clean_pdf:
	rm ${FILENAME}.pdf
clean_all: clean clean_pdf
```

## Batch file to compile on Windows
```powershell
set FILENAME=filename
set BIBNAME=filename
	
@echo off
cls
:question
echo.
echo 1. Compile the document
echo 2. Clean everything but keep the PDF
echo 3. Really clean EVERYTHING
set /p choix=What do you want to do (1/2/3)? :

if /I "%choix%"=="1" (goto :compile)
if /I "%choix%"=="2" (goto :clean)
if /I "%choix%"=="3" (goto :clean_all)
goto question

:compile
pdflatex %FILENAME%.tex
bibtex %BIBNAME%
pdflatex %FILENAME%.tex
pdflatex %FILENAME%.tex
goto end

:clean
del %FILENAME%.aux
del %BIBNAME%.bbl
del %BIBNAME%.blg
del %FILENAME%.log
goto end

:clean_all
del %FILENAME%.aux
del %BIBNAME%.bbl
del %BIBNAME%.blg
del %FILENAME%.log
del %FILENAME%.pdf
goto end

:end
echo Thanks for using this tool :)
```
