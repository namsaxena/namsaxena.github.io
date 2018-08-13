---
layout: post
title: "Export Stata Output to Latex"
date: 2018-08-13
---

Following is a sample do-file that exports Stata output to a pretty Latex table.

Exporting Stata output to Latex tables is relatively straightforward. If you want to make a complicated table in Latex with output from various
regressions or report summary statistics for many variables, by treatment assignment, you can use the following do-file as a starting point and 
modify it according to your needs. I have shared this code with RAs at the National Bureau of Economic Research because I think it's a
painless way to make tables.

Declaration: This code did not originate with me. I received a bare-bones template of how to export Stata macros into Latex several moons ago so I am 
passing it on as public service.

Notice that I use a semicolon (;) delimiter.

```
#delimit (;) ;
clear all ;
pause on ;
set more off ;
cap log close ;

		/* Code by: Namrata Narain 
					nnarains@gmail.com
					National Bureau of Economic Research
		*/	
			
	***** begin preamble ***** ;
	
	* path names ;
	global root "C:\XXX\gptw" ;
	global do_root "C:\XXX\gptw\dofiles" ;
	global log_root "C:\XXX\gptw\logfiles" ;
	global dest "C:\XXX\gptw\dest" ;
	global working "C:\XXX\gptw\working" ;
	global output "C:\XXX\gptw\output\TablesFigures_08032018" ;
	cd "$root" ;
	
	* log files ;
	loc filename "Table1a_SummStats" ;
	local d = td(`c(current_date)') ;
	local start_time = tc(`c(current_time)') ;
	local tstamp = string(year(`d'))+string(month(`d'))
		+string(day(`d'))+"_"+string(hh(`start_time')) + "nn" ;
	
	log using "$log_root\log_`filename'_`tstamp'.txt", text replace ;
  
  ***** end preamble ***** ;
	
	***** Set Up *************
	************************** ;
	
	use "$dest\gptw_allvars_built.dta", clear ;
	
	keep firm indassign totalmat fullmat avgmat ;
	
	gen freq = 1 ;
	foreach var in totalmat fullmat avgmat { ;
		gen nm_`var' = 1 if !mi(`var') ;
	} ;
	
	collapse (sum) freq nm_totalmat nm_fullmat nm_avgmat 
		(mean) totalmat fullmat avgmat, by(indassign) ;
	
	rename indassign industry ;
	
	sum freq ;
	gen perc = -99 ;
	replace perc = freq/r(sum) ;
	replace perc = perc*100 ;
	
	***** WRITE TO TABLE ***** 
	************************** ;
	
	file open fh using "$output/`filename'.tex", write replace ;

	file write fh "\documentclass[portrait 10pt]{article}" _n
		"\pagestyle{empty}" _n
		"\usepackage[utf8]{inputenc}" _n  
		"\usepackage{pslatex}" _n
		"\usepackage{longtable}"_n
		"\usepackage{multirow}" _n
		"\usepackage[left=.5cm,top=3cm,right=.5cm,bottom=3cm,nohead,nofoot, portrait]{geometry}" _n
		"\usepackage{dcolumn}" _n
		"\newcolumntype{d}[0]{D{.}{.}{5}}" _n
		"\begin{document}" _n
		"\begin{center}" _n
		"\begin{longtable}{l l c c c c c}" _n 
		"\multicolumn{7}{c}{\makebox[0pt]{Table 1. Summary Statistics}} \\ " _n
		"\hline" _n
		" \multicolumn{1}{l}{} &  &  & Total Days & Fully Paid & Average Days \\" _n
		" \multicolumn{1}{l}{Industry} & Number & Percent & Maternity & Maternity & Maternity \\" _n
		"\hline \hline" _n
		"\hline  \\ " _n ;

		
	file write fh "\emph{Panel A: Industry Level} \\" _n ;	
	levelsof industry,loc(inds) ;
	
	foreach ind in `inds' { ;
		
		sum freq if industry == "`ind'" ;
		loc number = r(mean) ;
		loc number : di %3.0f `number' ; 
		
		sum perc if industry == "`ind'" ;
		loc perc = r(mean) ;
		loc perc : di %4.3f `perc' ;
		
		sum totalmat if industry == "`ind'" ;
		loc totmom = r(mean) ;
		loc totmom : di %3.0f `totmom' ;
		
		sum fullmat if industry == "`ind'" ;
		loc fullmom = r(mean) ;
		loc fullmom : di %3.0f `fullmom' ;
		
		sum avgmat if industry == "`ind'" ;
		loc avgmom = r(mean) ;
		loc avgmom : di %3.0f `avgmom' ;
		
		file write fh "\hspace{20pt} `ind' & `number' & `perc' & `totmom' & `fullmom' & `avgmom'  \\" _n ;
	} ;
	
		* totals ;
		sum freq ;
		loc number = r(sum) ;
		loc number : di %3.0f `number' ;
		
		sum perc ;
		loc perc = r(sum) ;
		loc perc : di %3.0f `perc' ;
		
		file write fh "\\" _n ;
		file write fh "\hspace{20pt}\emph{Total} & `number' & `perc' & & & \\" _n ;
		
		* print the number of firms not missing information ;
		sum nm_totalmat ;
		loc nm_totmom = r(sum) ;
		
		sum nm_fullmat ;
	 	loc nm_fullmom = r(sum) ;
		
		sum nm_avgmat ;
		loc nm_avgmom = r(sum) ;
		
		file write fh "\hspace{20pt}\emph{Total Non-Missing} & & & `nm_totmom' & `nm_fullmom' & `nm_avgmom'  \\" _n ;
		file write fh "\\" _n ;
		
	file write fh
		"\hline \hline " _n
		"\end{longtable}" _n
		"\begin{tabular}{ p{5in} }" _n
		"\footnotesize" _n
		"Notes: This table provides industry-level summary statistics of data
		scraped from www.greatplacetowork.com. Industry assignments
		are made according to SIC codes that I assigned to each company 
		manually.\\" 
		"\end{tabular}" _n
		"\end{center}" _n
		"\end{document}" _n ;

	file close fh;
	cap erase "$output\`filename'.aux" ; 
	cap erase "$output\`filename'.log" ; 
	cap erase "$output\`filename'.pdf" ; 
	cap erase "$output\`filename'.synctex.gz " ; 

```
