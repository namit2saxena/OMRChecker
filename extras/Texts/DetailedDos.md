#<to be reformatted later>
0. Normalize the brightness: Run on 10-15 scans of empty OMRs for threshold tuning
> Can you plot the histogram on single image?
--> Yeah, that's skewed! Shud use the full spectrum! 
> Is it bad? Actually the black is on the left, white is on the right thats good.
> The gray is black too rn, it will gray out. 
> Gray= 127-->80-->53.82(BGLR_JE_03,healthy, maxB 42, avgW  in that case. Usually maxB ~30, avgW ~ 220)
> Black= 50-->48

#### Conclusions
O = Healthy
X = Bad
M = higherExt
m = lowerExt

Description			maxB avgB minW avgW 
Slight Shift		 O	   O	m	O
Risky But Works		 M 	   O 	m 	X  <--- JE_05 vs JE_04 Bangalore
							...
							...

Gap between gray and black as HEALTH/CONFIDENCE MEASURE?


** FOUND MAJOR CAUSE FOR TEMPLATE SHIFT:
	> It's not about scan being of a curved paper. The print itself is wrong. The xerox machine got the curved paper! Coz this is happening only with some xeroxes (no normal prints).
	== It's a proper scan of improper print 
	==> Akindle terms this 'Nonlinear distortion caused by document feed scanner' https://imgur.com/a/10qwL

## Curr Tasks
### 1) Run the outputs into omr detector.
	1.1 [X] Rescale Thresholds into 0-255 range
	1.2 [X] Show clr vals at the boxes.
	1.3 [X] Readjust threshold values
	1.4 [X] Show b4_after comparision

### 2) For multi marks: Local variance measure to determine multimarked. > Can you plot the threshold distribution?
	2.0 [ ]	 Nope, they gone- Gather 5 images on which current code detects false multi marks
		> Rather increase the THR to obtain multimarks
			thresholdRead_L =  145 # 110 # if kv else 120
			thresholdRead_R =  77 # 50 
			Found JE Bangalore Series to fail on this.

	2.1 [X]	Refactor Template to suit question-wise detection
	>> Main thing is: qNo,point,qType <- lets make a Q class _/
		// Derive template from ans? - nope. Some may want 1,2,3,4 as option numbers
		// Merge anskey into template? - nope. Why add this constraint? May fail at unforeseen situation?!
		>> NOPE: Create from answer key. // rather create ans key from template
	[X] Scale the coords onto high res image.
	[ ] Give feedback on adjusting template to make scaleRange near 1 


	2.2 [ ]	Refactor readResponse 


	2.3 [ ]	Implement local variance
	[X]	lets Plot the hist!
		[:28]> First make Q Class
		[:30]> Modify template code
		[:42]> As we're likely to use outer rect boxes, keep support for a qNoList for multi qs in the rect at gaps[1]
			### DESIGN CHOICE
			C1) Current template making 
				+Faster to make in ideal situations
				+Superset of below
				~More complex rects makeable in less data
				-Stubborn
				+Template fitting is robust - all or none
			C2) Individiual Question Coords
				+Simple
				~Code repetition --> Actually is healthy here as the region is active!
				-Slightly slower
				-More user clicks required
				+eliminates gaps[1]
				~On curvy papers this would disturb only some questions
				+Can be copy pasted in future <- seamlessness support
				+Class support easy <- for histograms!
			Verdict: C2 _/_/
			"""
			New Input:
			start coord, type for each question
			gapX, gapY be fixed for a Q type in the template -Nope-its also supported by core fn_/ _/ Supportable by fixed mini-templates later
			numX, numY removed and hardcoded.
			>> Need a Pt class to contain coord and read-value
			New Output
			Array of Q objects
			"""
		[+24 2:12]> Subplot Hist for each Q <- or maybe that candlestick one
			>> Finds that first bar of the histogram indicates correctly marked vals

(12 Sept 2018)
	2.4 [X] JSONify the template
	2.5 [X] Fit the template for mcq and rolls

(12 Sept 2018)
### 3) Plot for each question
	3.1 [X] Change readResponse to adapt to Q class
	3.2 [X] Make hist subplots
	3.3 [X] Make boxplot subplots _/_/
	3.4 [X] Record/Report the progress
		>> Durgapur_HE_03 is edgy case for Q10 - cant handle despite normalization, need more parameters! -> 'tone' of the image means more!!
		>> Got much better visualizations of the parameters now!
	... If there's any 3D data, think if using contours would help somewhere
	
	(11 Oct 2018)
	3.5 [X] Local threshold acc to this variance & mean: 
		Local == per-image
		M1: Set marginal thresholds per image! 
			--> Sorting, and taking min + 0.4*(diff) as blackTHR, min + 0.6*(diff) as whiteTHR, used in [2013](https://www.researchgate.net/publication/272729555_Cost_Effective_Optical_Mark_Reader) with scope for ambiguity detection i.e. separates partially erased marks(gray bubbles) too.
			--> But needs one more pass and much calculation, rather tuning from 'nearest threshold' might be better: 
			--> Margin for xerox would be really low, and Non-xerox still works fine
			--> At this config, this basically would simply reject xerox grays
			--> This rather does show the jump between black and gray!

		[X] M2: Just find the jumps instead of fixating on .4 and .6
			--> Would definitely separate into 3 :  black, gray and white
			--> Unless the jump isn't significant (Ambiguity), your function shall give the best jump psble still.
			--> Rather find only first jump. No ambiguity for plain gray (maybe later find middle jump for erased ones)
		
		Local == per-question
			--> This would be an overkill as image is already normalized.


(11 Oct 2018)
### 4) Check for more improvements: 
	4.1 [..] Nope- Its effect isn't significant as most image is white and even gray ones need to be closer to white. 
	Maybe **check this later** on getting mobile images.  CLAHE(Contrast Limited Adaptive Histogram Equalization) looks better than Normalization [2015](https://www.researchgate.net/publication/280776405_CheckIt_-_A_low_cost_mobile_OMR_system)

	4.2 [X] Nope, there may not be any white there(think multi marked) - Normalize only at Q-level!

	4.3 [X] Nope, slight shifted templates will have prob! -Change mean to gaussian in boxreading for finer results
	4.4 [X] (24 Mar 19) Explore uses of lowpass, highpass filters - maybe required for mobile images
			--> they are too simple to be useful. Some sophisticated DFT effect might be useful

(13 Oct 2018)
### 5) Testbench generation
	5.1 [X] Nope, too many images - rotate in steps of 2 degrees till 360
	5.2 [X] Nope, distortive: warpAffine = skewing -- change 2d perspective		
	5.3 [X] Learn about and use unittesting
	5.4 [X] Refresh about geometric transformations [ref](https://docs.opencv.org/3.0-beta/doc/py_tutorials/py_imgproc/py_geometric_transformations/py_geometric_transformations.html)
	5.5 [X] (14 Oct 2018) Rotation without clipping
	5.6 [X] Translation with clipping
	5.7 [X] Add background to modified image (getting boundary outer black mask would've been easier, but you chose to store points!)
	5.8 [X] ReversePerspective with clipping: get Points by rotating lines of rectangle in outward expanding way
	5.9 [ ] Images with more than 4 circles, Less than 4 ?!
	5.10 [ ] Brightness and contrast variations

### 6) Make note current code benchmarks 
	6.1 [X] Template match test: Perspective angle margins using blurred template matching
	6.1 [X] Full test results

(5 Nov 2018)
### 7) Find/Ideate and try out force aligning methods
	7.1 [X] Note Why it is needed - For 'proper scan of improper print' i.e. unevenly widened print. Also for slight template circle shifts due to inaccuracy there.
	7.2 Find what methods are used to do this
	7.3 [X] nope - Using contours for inner boxes 
	7.4 [X] Per-box averaged alignment ?!


(23 Mar 2019)
### 8) Refactor readResponse completely		
	8.1 [X] Simplify resp array generation
	8.2 [X] Simplify detection process
	8.3 [X] Minimize logging 

[X]	Work on full resolution first, then scale down at the end.


## Design Choices
	
### Function Means chart

#### Dev side
(Note: info old below. to be updated after decisions)

| Function         | Means             |             |                   |   			|
|------------------|-------------------|-------------|-------------------|--------------|
| Image Color  	   | [Grayscale]	   | RGB 	     | BGR 				 |   			|
| Choosing ROI     | 4 Opaque squares  | [4 circles] | Sidebars          |   Contours	|
| Setting Template | Individual Points | Q-wise      | [QRectangle-wise] |   			|
| Resolution       | [High scaledown]  | scaledown 	 | Original      	 |  Scaled Up   |
| Thresholding     | [per-Img marginal]| Tone-based* | Fixed             | 	 			|

* to be achieved

