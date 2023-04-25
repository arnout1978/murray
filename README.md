# murray
murray-code
//@version=4
study("Murrey Math Lines", title="MML", max_bars_back=500, overlay=true)



// Function outputs 1 when it's the first bar of the D/W/M/Y
is_newbar(res) =>
    ch = 0
    if(res == 'Y')
        t  = year(time('D'))
        ch := change(t) != 0 ? 1 : 0
    else
        t = time(res)
        ch := change(t) != 0 ? 1 : 0
    ch

// Rounding levels to min tick
nround(x) => 
    n = round(x / syminfo.mintick) * syminfo.mintick
//-- get inputs
frame = input(defval=64, title="Frame Size", type=input.integer, minval=8, maxval=256)
mult = input(defval=1.5, title="Frame Multiplier", type=input.float, minval=1.0, maxval=2.0, step=0.5)
wicks = input(defval=true, title="Ignore Wicks?")
show_historical_levels = input(title = "Show Historical Levels?",  defval = false,  type = input.bool)
extendOption = input(title="Line Extension", type=input.string,      options=["none", "left", "right", "both"], defval="none")
lineExtend = (extendOption == "left") ? extend.left : (extendOption == "right") ? extend.right : (extendOption == "both") ? extend.both : extend.none
show_level_value       = input(title = "Show Levels Value?",       defval = true,   type = input.bool)
show_current_levels    = input(title = "Show Current Levels",      defval = false,  type = input.bool)
//-- defines
logTen = log(10)
log8 = log(8)
log2 = log(2)
lookback = round(frame * mult)

//-- determine price values based on ignore wick selection
uPrice = wicks == true ? max(open, close) : high
lPrice = wicks == true ? min(open, close) : low

//-- find highest/lowest price over specified lookback
vLow = lowest(lPrice, lookback)
vHigh = highest(uPrice, lookback)
vDist = vHigh - vLow
//-- if low price is < 0 then adjust accordingly
tmpHigh = vLow < 0 ? 0 - vLow : vHigh
tmpLow = vLow < 0 ? 0 - vLow - vDist : vLow

//-- determine if price shift is in place
shift = vLow < 0 ? true : false

//-- calculate scale frame
sfVar = log(0.4 * tmpHigh) / logTen - floor(log(0.4 * tmpHigh) / logTen)
SR = tmpHigh > 25 ? 
   sfVar > 0 ? exp(logTen * (floor(log(0.4 * tmpHigh) / logTen) + 1)) : 
   exp(logTen * floor(log(0.4 * tmpHigh) / logTen)) : 
   100 * exp(log8 * floor(log(0.005 * tmpHigh) / log8))
nVar1 = log(SR / (tmpHigh - tmpLow)) / log8
nVar2 = nVar1 - floor(nVar1)
N = nVar1 <= 0 ? 0 : nVar2 == 0 ? floor(nVar1) : floor(nVar1) + 1

//-- calculate scale interval and temporary frame top and bottom
SI = SR * exp(-N * log8)
M = floor(1.0 / log2 * log((tmpHigh - tmpLow) / SI) + 0.0000001)
I = round((tmpHigh + tmpLow) * 0.5 / (SI * exp((M - 1) * log2)))

Bot = (I - 1) * SI * exp((M - 1) * log2)
Top = (I + 1) * SI * exp((M - 1) * log2)

//-- determine if frame shift is required
doShift = tmpHigh - Top > 0.25 * (Top - Bot) or Bot - tmpLow > 0.25 * (Top - Bot)

ER = doShift == true ? 1 : 0

MM = ER == 0 ? M : ER == 1 and M < 2 ? M + 1 : 0
NN = ER == 0 ? N : ER == 1 and M < 2 ? N : N - 1

//-- recalculate scale interval and top and bottom of frame, if necessary
finalSI = ER == 1 ? SR * exp(-NN * log8) : SI
finalI = ER == 1 ? round((tmpHigh + tmpLow) * 0.5 / (finalSI * exp((MM - 1) * log2))) : I
finalBot = ER == 1 ? (finalI - 1) * finalSI * exp((MM - 1) * log2) : Bot
finalTop = ER == 1 ? (finalI + 1) * finalSI * exp((MM - 1) * log2) : Top

//-- determine the increment
Increment = (finalTop - finalBot) / 8

//-- determine the absolute top
absTop = shift == true ? -(finalBot - 3 * Increment) : finalTop + 3 * Increment

//-- create our Murrey line variables based on absolute top and the increment
Plus38 = absTop
Plus28 = absTop - Increment
Plus18 = absTop - 2 * Increment
EightEight = absTop - 3 * Increment
SevenEight = absTop - 4 * Increment
SixEight = absTop - 5 * Increment
FiveEight = absTop - 6 * Increment
FourEight = absTop - 7 * Increment
ThreeEight = absTop - 8 * Increment
TwoEight = absTop - 9 * Increment
OneEight = absTop - 10 * Increment
ZeroEight = absTop - 11 * Increment
Minus18 = absTop - 12 * Increment
Minus28 = absTop - 13 * Increment
Minus38 = absTop - 14 * Increment

bars_sinse = 0
bars_sinse := is_newbar('D') ? 0 : bars_sinse[1] + 1
//-- plot the lines and we are done
vsm1_p = line.new(bar_index[min(bars_sinse, 300)], Plus38, bar_index, Plus38, color=color.white,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm1_p, extend=lineExtend)
vsm2_p = line.new(bar_index[min(bars_sinse, 300)], Plus28, bar_index, Plus28, color=color.red,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm2_p, extend=lineExtend)
vsm3_p = line.new(bar_index[min(bars_sinse, 300)], Plus18, bar_index, Plus18, color=color.red,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm3_p, extend=lineExtend)
vsm4_p = line.new(bar_index[min(bars_sinse, 300)], EightEight, bar_index, EightEight, color=color.aqua,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm4_p, extend=lineExtend)
vsm5_p = line.new(bar_index[min(bars_sinse, 300)], SevenEight, bar_index, SevenEight, color=color.orange,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm5_p, extend=lineExtend)
vsm6_p = line.new(bar_index[min(bars_sinse, 300)], SixEight, bar_index, SixEight, color=color.fuchsia,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm6_p, extend=lineExtend)
vsm7_p = line.new(bar_index[min(bars_sinse, 300)], FiveEight, bar_index, FiveEight, color=color.green,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm7_p, extend=lineExtend)
vsm8_p = line.new(bar_index[min(bars_sinse, 300)], FourEight, bar_index, FourEight, color=color.aqua,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm8_p, extend=lineExtend)
vsm9_p = line.new(bar_index[min(bars_sinse, 300)], ThreeEight, bar_index, ThreeEight, color=color.green,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm9_p, extend=lineExtend)
vsm10_p = line.new(bar_index[min(bars_sinse, 300)], TwoEight, bar_index, TwoEight, color=color.fuchsia,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm10_p, extend=lineExtend)
vsm11_p = line.new(bar_index[min(bars_sinse, 300)], OneEight, bar_index, OneEight, color=color.orange,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm11_p, extend=lineExtend)
vsm12_p = line.new(bar_index[min(bars_sinse, 300)], ZeroEight, bar_index, ZeroEight, color=color.aqua,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm12_p, extend=lineExtend)
vsm13_p = line.new(bar_index[min(bars_sinse, 300)], Minus18, bar_index, Minus18, color=color.green,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm13_p, extend=lineExtend)
vsm14_p = line.new(bar_index[min(bars_sinse, 300)], Minus28, bar_index, Minus28, color=color.green,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm14_p, extend=lineExtend)
vsm15_p = line.new(bar_index[min(bars_sinse, 300)], Minus38, bar_index, Minus38, color=color.white,   style =  line.style_dotted, extend = extend.none,width = 2)
line.set_extend(id=vsm15_p, extend=lineExtend)

//Delete

if (not is_newbar('D') or not show_historical_levels)
    //line.delete(vpp_p[1])
    line.delete(vsm1_p[1]) 
    line.delete(vsm2_p[1])  
    line.delete(vsm3_p[1])  
    line.delete(vsm4_p[1])
    line.delete(vsm5_p[1]) 
    line.delete(vsm6_p[1]) 
    line.delete(vsm7_p[1]) 
    line.delete(vsm8_p[1]) 
    line.delete(vsm9_p[1]) 
    line.delete(vsm10_p[1]) 
    line.delete(vsm11_p[1]) 
    line.delete(vsm12_p[1]) 
    line.delete(vsm13_p[1]) 
    line.delete(vsm14_p[1])
    line.delete(vsm15_p[1])
   
//Labelling
label_vsm1 = label.new(bar_index, Plus38, text=show_level_value ? ("                                                        +3/8 Imminent Bearish reversal -- " + " " + tostring(nround(Plus38))) : "+3/8 Imminent Bearish reversal -- ", style= label.style_none, textcolor = color.white)
label_vsm2 = label.new(bar_index, Plus28, text=show_level_value ? ("                                                        +2/8 Extreme Overshoot -- " + " " + tostring(nround(Plus28))) : "+2/8 Extreme Overshoot -- ", style= label.style_none, textcolor = color.red)
label_vsm3 = label.new(bar_index, Plus18, text=show_level_value ? ("                                                        +1/8 Overshoot -- " + " " + tostring(nround(Plus18))) : "+1/8 Overshoot -- ", style= label.style_none, textcolor = color.red)
label_vsm4 = label.new(bar_index, EightEight, text=show_level_value ? ("                                                        8/8 Ultimate resistance -- " + " " + tostring(nround(EightEight))) : "8/8 Ultimate resistance -- ", style= label.style_none, textcolor = color.aqua)
label_vsm5 = label.new(bar_index, SevenEight, text=show_level_value ? ("                                                        7/8 Weak, Stop & Reverse -- " + " " + tostring(nround(SevenEight))) : "7/8 Weak, Stop & Reverse -- ", style= label.style_none, textcolor = color.orange)
label_vsm6 = label.new(bar_index, SixEight, text=show_level_value ? ("                                                        6/8 Strong pivot reverse -- " + " " + tostring(nround(SixEight))) : "6/8 Strong pivot reverse -- ", style= label.style_none, textcolor = color.fuchsia)
label_vsm7 = label.new(bar_index, FiveEight, text=show_level_value ? ("                                                        5/8 Top of trading range -- " + " " + tostring(nround(FiveEight))) : "5/8 Top of trading range -- ", style= label.style_none, textcolor = color.green)
label_vsm8 = label.new(bar_index, FourEight, text=show_level_value ? ("                                                        4/8 Major S/R pivot point -- " + " " + tostring(nround(FourEight))) : "4/8 Major S/R pivot point -- ", style= label.style_none, textcolor = color.aqua)
label_vsm9 = label.new(bar_index, ThreeEight, text=show_level_value ? ("                                                        3/8 Bottom of trading range -- " + " " + tostring(nround(ThreeEight))) : "3/8 Bottom of trading range -- ", style= label.style_none, textcolor = color.green)
label_vsm10 = label.new(bar_index, TwoEight, text=show_level_value ? ("                                                        2/8 Strong, Pivot, reverse -- " + " " + tostring(nround(TwoEight))) : "2/8 Strong, Pivot, reverse -- ", style= label.style_none, textcolor = color.fuchsia)
label_vsm11 = label.new(bar_index, OneEight, text=show_level_value ? ("                                                        1/8 Weak, Stop & Reverse -- " + " " + tostring(nround(OneEight))) : "1/8 Weak, Stop & Reverse -- ", style= label.style_none, textcolor = color.orange)
label_vsm12 = label.new(bar_index, ZeroEight, text=show_level_value ? ("                                                        0/8 Ultimate Support-- " + " " + tostring(nround(ZeroEight))) : "0/8 ultimate Support -- ", style= label.style_none, textcolor = color.aqua)
label_vsm13 = label.new(bar_index, Minus18, text=show_level_value ? ("                                                        -1/8 Oversold-- " + " " + tostring(nround(Minus18))) : "-1/8 Oversold -- ", style= label.style_none, textcolor = color.green)
label_vsm14 = label.new(bar_index, Minus28, text=show_level_value ? ("                                                        -2/8 Extreme Oversold-- " + " " + tostring(nround(Minus28))) : "-2/8 Extreme Oversold -- ", style= label.style_none, textcolor = color.green)
label_vsm15 = label.new(bar_index, Minus38, text=show_level_value ? ("                                                        -3/8 Imminent Bullish reversal-- " + " " + tostring(nround(Minus38))) : "-3/8 Imminent Bullish reversal -- ", style= label.style_none, textcolor = color.white)
//Label delete
if (not is_newbar('D') or not show_historical_levels)
    label.delete(label_vsm1[1])
    label.delete(label_vsm2[1])
    label.delete(label_vsm3[1])
    label.delete(label_vsm4[1])
    label.delete(label_vsm5[1])
    label.delete(label_vsm6[1])
    label.delete(label_vsm7[1])
    label.delete(label_vsm8[1])
    label.delete(label_vsm9[1])
    label.delete(label_vsm10[1])
    label.delete(label_vsm11[1])
    label.delete(label_vsm12[1])
    label.delete(label_vsm13[1])
    label.delete(label_vsm14[1])
    label.delete(label_vsm15[1])



////////////
// ALERTS //
ot = Plus38 > Plus38[1] or Plus38 < Plus38[1] or Plus28 > Plus28[1] or Plus28 < Plus28[1] or Plus18 > Plus18[1] or Plus18 < Plus18[1] or EightEight > EightEight[1] or SevenEight > SevenEight[1] or SevenEight < SevenEight[1] or SixEight > SixEight[1] or SixEight < SixEight[1] or FiveEight > FiveEight[1] or FiveEight < FiveEight[1] or FourEight > FourEight[1] or FourEight < FourEight[1] or ThreeEight > ThreeEight[1] or ThreeEight < ThreeEight[1] or TwoEight > TwoEight[1] or TwoEight < TwoEight[1] or OneEight > OneEight[1] or OneEight < OneEight[1] or ZeroEight > ZeroEight[1] or ZeroEight < ZeroEight[1] or Minus18 > Minus18[1] or Minus18 < Minus18[1] or Minus28 > Minus28[1] or Minus38 > Minus38[1] or Minus38 < Minus38[1]
alertcondition(ot, title="Alert me when Octaves Change!", message="Octaves Change - Alert!")
 
    
alertcondition(not is_newbar('D') and cross(close, Plus38), "Cross +3/8",  "Cross +3/8")
alertcondition(not is_newbar('D') and cross(close, Plus28), "Cross +2/8", "Cross +2/8")
alertcondition(not is_newbar('D') and cross(close, Plus18), "Cross +1/8", "Cross +1/8")
alertcondition(not is_newbar('D') and cross(close, EightEight), "Cross 8/8", "Cross 8/8")
alertcondition(not is_newbar('D') and cross(close, SevenEight), "Cross 7/8", "Cross 7/8")
alertcondition(not is_newbar('D') and cross(close, SixEight), "Cross 6/8", "Cross 6/8")
alertcondition(not is_newbar('D') and cross(close, FiveEight), "Cross 5/8", "Cross 5/8")
alertcondition(not is_newbar('D') and cross(close, FourEight), "Cross 4/8", "Cross 4/8")
alertcondition(not is_newbar('D') and cross(close, ThreeEight), "Cross 3/8", "Cross 3/8")
alertcondition(not is_newbar('D') and cross(close, TwoEight), "Cross 2/8", "Cross 2/8")
alertcondition(not is_newbar('D') and cross(close, OneEight), "Cross 1/8", "Cross 1/8")
alertcondition(not is_newbar('D') and cross(close, ZeroEight), "Cross 0/8", "Cross 0/8")
alertcondition(not is_newbar('D') and cross(close, Minus18), "Cross -1/8", "Cross -1/8")
alertcondition(not is_newbar('D') and cross(close, Minus28), "Cross -2/8", "Cross -2/8")
alertcondition(not is_newbar('D') and cross(close, Minus38), "Cross -3/8", "Cross -3/8")




alertcondition(not is_newbar('D') and crossover(close, Plus38), "Crossover +3/8",  "Crossover +3/8")
alertcondition(not is_newbar('D') and crossover(close, Plus28), "Crossover +2/8", "Crossover +2/8")
alertcondition(not is_newbar('D') and crossover(close, Plus18), "Crossover +1/8", "Crossover +1/8")
alertcondition(not is_newbar('D') and crossover(close, EightEight), "Crossover 8/8", "Crossover 8/8")
alertcondition(not is_newbar('D') and crossover(close, SevenEight), "Crossover 7/8", "Crossover 7/8")
alertcondition(not is_newbar('D') and crossover(close, SixEight), "Crossover 6/8", "Crossover 6/8")
alertcondition(not is_newbar('D') and crossover(close, FiveEight), "Crossover 5/8", "Crossover 5/8")
alertcondition(not is_newbar('D') and crossover(close, FourEight), "Crossover 4/8", "Crossover 4/8")
alertcondition(not is_newbar('D') and crossover(close, ThreeEight), "Crossover 3/8", "Crossover 3/8")
alertcondition(not is_newbar('D') and crossover(close, TwoEight), "Crossover 2/8", "Crossover 2/8")
alertcondition(not is_newbar('D') and crossover(close, OneEight), "Crossover 1/8", "Crossover 1/8")
alertcondition(not is_newbar('D') and crossover(close, ZeroEight), "Crossover 0/8", "Crossover 0/8")
alertcondition(not is_newbar('D') and crossover(close, Minus18), "Crossover -1/8", "Crossover -1/8")
alertcondition(not is_newbar('D') and crossover(close, Minus28), "Crossover -2/8", "Crossover -2/8")
alertcondition(not is_newbar('D') and crossover(close, Minus38), "Crossover -3/8", "Crossover -3/8")


alertcondition(not is_newbar('D') and crossunder(close, Plus38), "Crossunder +3/8",  "Crossunder +3/8")
alertcondition(not is_newbar('D') and crossunder(close, Plus28), "Crossunder +2/8", "Crossunder +2/8")
alertcondition(not is_newbar('D') and crossunder(close, Plus18), "Crossunder +1/8", "Crossunder +1/8")
alertcondition(not is_newbar('D') and crossunder(close, EightEight), "Crossunder 8/8", "Crossunder 8/8")
alertcondition(not is_newbar('D') and crossunder(close, SevenEight), "Crossunder 7/8", "Crossunder 7/8")
alertcondition(not is_newbar('D') and crossunder(close, SixEight), "Crossunder 6/8", "Crossunder 6/8")
alertcondition(not is_newbar('D') and crossunder(close, FiveEight), "Crossunder 5/8", "Crossunder 5/8")
alertcondition(not is_newbar('D') and crossunder(close, ThreeEight), "Crossunder 4/8", "Crossunder 4/8")
alertcondition(not is_newbar('D') and crossunder(close, FourEight), "Crossunder 3/8", "Crossunder 3/8")
alertcondition(not is_newbar('D') and crossunder(close, TwoEight), "Crossunder 2/8", "Crossunder 2/8")
alertcondition(not is_newbar('D') and crossunder(close, OneEight), "Crossunder 1/8", "Crossunder 1/8")
alertcondition(not is_newbar('D') and crossunder(close, ZeroEight), "Crossunder 0/8", "Crossunder 0/8")
alertcondition(not is_newbar('D') and crossunder(close, Minus18), "Crossunder -1/8", "Crossunder -1/8")
alertcondition(not is_newbar('D') and crossunder(close, Minus28), "Crossunder -2/8", "Crossunder -2/8")
alertcondition(not is_newbar('D') and crossunder(close, Minus38), "Crossunder -3/8", "Crossunder -3/8")
