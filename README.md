# Reevaluacion
clear


global enaho_carla ="C:\Users\csolis\Documents\Algoritmo PCA\Bases de datos\Bases intermedias"
global base_enaho  ="C:\Users\csolis\Documents\Algoritmo PCA\Bases de datos\Insumos\ENAHO\"
global base_trabj  ="C:\Users\csolis\Documents\Algoritmo PCA\Bases de datos\Bases intermedias"

/*
global enaho_carla ="D:\MIDIS 2015\METODOLOGÍA MIDIS 2015\2. Implementación\bases\"
global base_enaho  ="G:\BASES DE DATOS\ENAHO\"
global base_trabj  ="D:\MIDIS 2015\METODOLOGÍA MIDIS 2015\3. Reevaluación\1. indicadores\bases trab\"
*/
cd "$base_trabj"
set more off


* Desastres naturales y otros shocks
*----------------------------------------

foreach x of numlist 2007(1)2014{

use "$base_enaho\`x'\enaho01b-`x'-2", clear
keep  mes conglome vivienda hogar codperso codinfor p40_1 p40_2 p40_3 p40_4 p40_5 p40_6 p40_7 p40_8
tab1 p40_1 p40_2 p40_3 p40_4 p40_5 p40_6 p40_7 p40_8
gen nanio=`x'
destring conglome vivienda hogar nanio, replace
save "$base_trabj\enaho_shock_`x'", replace
}


use "$base_trabj\enaho_shock_2007", replace
foreach x of numlist 2008(1)2014 {
append using "$base_trabj\enaho_shock_`x'"
}

drop mes
save "$base_trabj\desastre_shock", replace

*loc base_empleo = "$base_enaho\\`ano'\enaho01a-`ano'-500.dta"
*loc base_miemb  = "$base_enaho\\`ano'\enaho01-`ano'-200.dta"

foreach x of numlist 2007(1)2014{

use conglome vivienda hogar codperso p204 p203 p207 p209 p208a using "$base_enaho\`x'\enaho01-`x'-200", clear
merge 1:1 conglome vivienda hogar codperso using "$base_enaho\`x'\enaho01a-`x'-500", keepusing(p507)

gen indep=((p203==1|p203==2)& p507==2)
gen depen=((p203==1|p203==2)& (p507==3|p507==4))

bysort conglome vivienda hogar :egen trab_indep=max(indep)
bysort conglome vivienda hogar :egen trab_depen=max(depen)

keep if p203==1
gen nanio=`x'
destring conglome vivienda hogar nanio, replace
keep conglome vivienda hogar nanio trab_indep trab_depen

save "$base_trabj\empleo_`x'",replace
}

use "$base_trabj\empleo_2007", clear
foreach x of numlist 2008(1)2014 {
append using "$base_trabj\empleo_`x'"
erase "$base_trabj\empleo_`x'.dta"
}

save "$base_trabj\empleo", replace


/*
local ano="2014"
loc base_equi   = "$base_enaho\\`ano'\enaho01-`ano'-612.dta"
loc base_agro   = "$base_enaho\\`ano'\enaho02-`ano'-2000.dta"
loc base_agro2  = "$base_enaho\\`ano'\enaho02-`ano'-2000a.dta"
loc base_redes  = "$base_enaho\\`ano'\enaho01-`ano'-800a.dta"
loc base_empleo = "$base_enaho\\`ano'\enaho01a-`ano'-500.dta"
loc base_miemb  = "$base_enaho\\`ano'\enaho01-`ano'-200.dta"
/

use "`base_empleo'"
y*/
 
foreach x in 2007 2008 2009 2010 2011 2012 2013 2014{
*cd "$base_trabj"
*
local ano="`x'"

loc base_equi  = "$base_enaho\\`ano'\enaho01-`ano'-612.dta"
loc base_agro  = "$base_enaho\\`ano'\enaho02-`ano'-2000.dta"
loc base_agro2 = "$base_enaho\\`ano'\enaho02-`ano'-2000a.dta"
loc base_redes = "$base_enaho\\`ano'\enaho01-`ano'-800a.dta"
loc base_empleo = "$base_enaho\\`ano'\enaho01a-`ano'-500.dta"
loc base_miemb  = "$base_enaho\\`ano'\enaho01-`ano'-200.dta"
cd "$base_trabj"

/*Empleo
*=========

use conglome vivienda hogar codperso p204 p203 p207 p209 p208a using "`base_miemb'", clear
merge 1:1 conglome vivienda hogar codperso using "`base_empleo'"

gen indep=((p203==1|p203==2)& p507==2)
gen depen=((p203==1|p203==2)& (p507==3|p507==4))

by conglome vivienda hogar:egen trab_indep=max(indep)
by conglome vivienda hogar:egen trab_depen=max(depen)

keep if p203==1
keep conglome vivienda hogar trab_indep trab_depen
save "empleo`x'",replace
*/

* Agricultura
*============

use "`base_agro'", clear
use conglome vivienda hogar p20002b1 p20002b2 p20002b3 using "`base_agro'", clear
*p20002b1:  ¿Cual es el área total de la explotación agropecuaria: Propia que trabaja actualmente? (incluye barbecho,descanso,etc.)
*p20002b2:  ¿cuál es el área total de la explotación agropecuaria:Propia que alquila, presta o cede a otros? 
*p20002b3:  ¿cuál es el área total de la explotación agropecuaria:Propia que alquila, recibe o trabaja de otros?
gen a_o="`x'"
collapse (sum) p20002b1 p20002b2 p20002b3, by(conglome vivienda hogar a_o)
save "temp_agro`x'",replace

use conglome vivienda hogar p2005b p2005f1 p2005f2 p2005f3 p2005f4 using "`base_agro2'", clear
*p2005b  extensión de la parcela (hectáreas)
*p2005f1 ¿el tipo de riego es :tecnificado?
*p2005f2 ¿el tipo de riego es :por gravedad?
*p2005f3 ¿el tipo de riego es :secano?
*p2005f4 ¿el tipo de riego es :pozo/agua subterránea? 

sort conglome vivienda hogar
by conglome vivienda hogar: gen parcela_total=sum(p2005b)
gen x1=(p2005f1!=0 & p2005f1!=.)
gen x2=(p2005f2!=0 & p2005f2!=.)
gen x3=(p2005f3!=0 & p2005f3!=.)
gen x4=(p2005f4!=0 & p2005f4!=.)
egen xtot=rsum(x1 x2 x3 x4)
replace x1=x1/xtot*p2005b
replace x2=x2/xtot*p2005b
replace x3=x3/xtot*p2005b
replace x4=x4/xtot*p2005b

label var x1 "%tierra con riego tecnificado"
label var x2 "%tierra con riego gravedad"
label var x3 "%tierra con riego secano"
label var x4 "%tierra con riego pozo,agua subt"

collapse (sum) p2005b x1 x2 x3 x4, by(conglome vivienda hogar)
save  "temp_agro`x'_2",replace
merge 1:1 conglome vivienda hogar using "temp_agro`x'", nogen
*merge 1:1 conglome vivienda hogar using "empleo`x'", nogen
gen hh_actagro=1
egen tierra_total=rsum(p20002b1 p20002b2 p20002b3)
save "agro`x'.dta", replace
erase "temp_agro`x'_2.dta"
erase "temp_agro`x'.dta"
clear

/* Equipamiento del hogar
*=======================

use conglome vivienda hogar p612n p612 p612a ubigeo dominio estrato using "`base_equi'", clear
*p612n equipamiento del hogar
*p612  ¿su hogar tiene?
*p612a ¿cuantos tiene?
reshape wide p612 p612a, i(conglome vivienda hogar ubigeo dominio estrato) j(p612n)
forv y=1/26{
replace p612a`y'=0 if p612`y'==0
}
gen a_o="`x'"
save "equi`x'",replace
clear
*/

* Redes
*========

use conglome vivienda hogar p801_*  using "`base_redes'", clear
forv y=1/15{
gen a`y'=(p801_`y'==`y')
}
gen a_o="`x'"
save "redes`x'.dta", replace

*merge 1:1 conglome vivienda hogar using "equi`x'.dta",nogen
merge 1:1 conglome vivienda hogar using "agro`x'.dta",nogen

save "base_`x'.dta", replace

}

cd "$base_trabj"
set more off
clear
use "base_2007.dta", clear
foreach x of numlist 2008(1)2014 {
append using "base_`x'.dta"
}

rename a_o nanio
destring conglome vivienda hogar nanio, replace
merge 1:1 conglome vivienda hogar nanio using "$enaho_carla\ENAHO_TOTA1L_Puntaje.dta"
keep if _m==3
drop _m
merge 1:1 conglome vivienda hogar nanio using "desastre_shock"
keep if _m==3
drop _m
merge 1:1 conglome vivienda hogar nanio using "empleo"
keep if _m==3
drop _m
label var p20002b1  "área total de la explotación agropecuaria: Propia que trabaja actualmente (incluye barbecho,descanso,etc.)"
label var p20002b2  "área total de la explotación agropecuaria:Propia que alquila, presta o cede a otros" 
label var p20002b3  "cuál es el área total de la explotación agropecuaria:Propia que alquila, recibe o trabaja de otros"

recode hh_actagro (.=0)

egen redes=rsum (a1 a2 a3 a4 a5 a6 a7 a8 a9 a10 a11 a12 a13 a14 a15)
gen desastre=(p40_6==1)


save "ENAHO_TOTAL_V2", replace
clear

use "$enaho_carla\base_miembros_rename.dta", clear
destring conglome vivienda hogar,replace
merge 1:1 conglome vivienda hogar codperso a_o using "$enaho_carla\base_miembros_rename_2014_imputado.dta" 
gen agropec=(c5==1|c5==2) & d2>13 & d2<99
gen x=(d2>13 & d2<99)
gen depend_no_agro=(c4==1 & (c5!=1 & c5!=2 & c5!=.))& d2>13 & d2<99
gen indep=(c4==2)& d2>13 & d2<99

gen anoeduc=. 
replace anoeduc =0          if (c2_i==1 | c2_i==2)		
replace anoeduc =c3_i         if (c2_i==3)
replace anoeduc =c3_i + 6     if (c2_i==4)
replace anoeduc =c3_i + 11    if (c2_i==5 | c2_i==6 )
replace anoeduc =c3_i + 16    if (c2_i==7)

gen ano_edu=anoeduc if d2>21 & d2<99


collapse (sum) agropec x depend_no_agro indep (mean) anoeduc ano_edu , by(conglome vivienda hogar a_o) 
gen porc_agro=agropec/x
label var porc_agro "% de PEA en Act. Agropecuarias"
save nuevas_variables,replace


use "ENAHO_TOTAL_V2",clear
rename nanio a_o 
merge 1:1 conglome vivienda hogar a_o using nuevas_variables 

saveold "ENAHO_TOTAL_V2",replace
clear
