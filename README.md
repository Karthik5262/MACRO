layout form !!GCSTools
	Title 'Move / Rotate Elements'
	frame .frame0 'Load Elements from' at x0
		button .add1 'CE' callback |!this.AddElements('CE')|
		-- button .add2 'Search' callback |!this.AddElements('Search')|
		button .clear 'Clear' callback |!this.items.clear()|
		button .reset 'Reset' at x 32 callback |!this.Reset()|
		button .refr 'Refresh' at x40 callback |!this.Refresh()|
	exit
	frame .frame1 'List is Empty'  anchor right + left + top at x0 ymax
		list  .items callback '!this.GotoCE()'  at xmin ymin  anchor all width 45 height 25
	exit
	frame .frame2 'Move / Rotate Elements' at x0 ymax
		rToggle .movesel |Move| call |!this.Toggle()| tagwidth 5
		rToggle .rotasel |Rotate| call |!this.Toggle()| tagwidth 5
		text .zdir  |Axis| tagwidth 4 call |!this.ParseDir()| width 6 is string 
		text .roteval  |Deg| tagwidth 4 width 6 is Real format !!angleFmt
		button .pick 'Pick Pos' callback |!this.PickPOS()|
		text .eastdist  |E| tagwidth 2 at x0 ymax + 0.3 call || width 10 is REAL format !!distanceFmt
		text .northdist  |N| tagwidth 2 at xmax ymin.eastdist call || width 10 is REAL format !!distanceFmt
		text .updist  |U| tagwidth 2 at xmax ymin.eastdist call || width 10 is REAL format !!distanceFmt
		button .action 'Move All' callback |!this.RunAction()|
	exit
Exit

-------------------------------------------------------------------------------------------------------------------
Define Method .GCSTools()
-------------------------------------------------------------------------------------------------------------------

!headings = split('Name,Owner',',')
!this.items.setheadings(!headings)
!this.eastdist.val = 0
!this.northdist.val = 0
!this.updist.val = 0
!this.roteval.val = 180
!this.movesel.val = true
!this.zdir.visible = false
!this.pick.visible = false
!this.roteval.visible = false
!this.zdir.val = 'U WRT/*'
!this.action.tag = 'Move All'

Endmethod

-------------------------------------------------------------------------------------------------------------------
Define Method .Reset()
-------------------------------------------------------------------------------------------------------------------
!this.frame1.tag =  'List is Empty'
!this.items.clear()
!this.GCSTools()

Endmethod

-------------------------------------------------------------------------------------------------------------------
Define Method .Refresh()
-------------------------------------------------------------------------------------------------------------------

kill !!GCSTools
SHOW !!GCSTools

Endmethod
-------------------------------------------------------------------------------------------------------------------
Define Method .ParseDir()
-------------------------------------------------------------------------------------------------------------------
!dir = object Direction(!this.zdir.val)
Handle (2,813)
	!this.zdir.val = 'U WRT/*'
	return error1 'Invalid Direction...'
Endhandle
!this.zdir.val = !dir.string()

Endmethod

-------------------------------------------------------------------------------------------------------------------
Define Method .PickPOS()
-------------------------------------------------------------------------------------------------------------------

var !ref REF OF ID@
Handle Any
	return
Endhandle
!pos = !ref.dbref().POS.wrt(world)
!this.eastdist.val = !pos.east
!this.northdist.val = !pos.north
!this.updist.val = !pos.up


Endmethod

-------------------------------------------------------------------------------------------------------------------
Define Method .Toggle()
-------------------------------------------------------------------------------------------------------------------

!this.zdir.visible = !this.rotasel.val
!this.pick.visible = !this.rotasel.val
!this.roteval.visible = !this.rotasel.val
if !this.movesel.val then
	!this.action.tag = 'Move All'
Else
	!this.action.tag = 'Rotate All'
Endif

Endmethod

-------------------------------------------------------------------------------------------------------------------
Define Method .RunAction()
-------------------------------------------------------------------------------------------------------------------
!this.ParseDir()
if !this.movesel.val then
	!this.MoveItems()
Else
	!this.RotateItems()
Endif

Endmethod

-------------------------------------------------------------------------------------------------------------------
Define Method .GotoCE()
-------------------------------------------------------------------------------------------------------------------

!ce = !this.items.selection('rtext')
$!ce
Handle Any
	!this.reset()
	return error1 'Invalid Item selected, List cleared'
Endhandle

Endmethod

-------------------------------------------------------------------------------------------------------------------
Define Method .InitCheck()
-------------------------------------------------------------------------------------------------------------------
if !this.eastdist.val.unset() or !this.northdist.val.unset() or !this.updist.val.unset() then
	return error1 'Input values missing...'
Endif

!elems = !this.items.rtext
if !elems.size() eq 0 then
	return error1 'No Elements in the list'
Endif

Endmethod

-------------------------------------------------------------------------------------------------------------------
Define Method .RotateItems()
-------------------------------------------------------------------------------------------------------------------
!this.InitCheck()

!dt = object datetime()
!date = !dt.string().replace(':','-').replace(' ','_') + '.log'
!log = Object File('%pdmsuser%\Move_$!date')
!logdata = Array()
!logdata.append('Report generated at ' + !dt.string())
!logdata.append(!this.frame1.tag)
!elems = !this.items.rtext
!logdata.append('###############################################################################################')
!logdata.appendarray(!elems)
!logdata.append('###############################################################################################')
!logdata.append('ROTATED THROUGH POSITION E $!this.eastdist.val N $!this.northdist.val U $!this.updist.val WRT/* ABOUT $!this.zdir.val BY $!this.roteval.val')
!tot = !elems.size()
if !!alert.confirm('Press Yes to confirm rotation of $!tot Elements from the mentioned position as Origin?') eq 'NO' then
	return
Endif
Do !x values !elems
	$!x
	-- !func = Func
	-- if !func eq 'Rotated' then
		-- !logdata.append('Skipped $!!CE.namtyp')
		-- skip
	-- Endif
	!pos = !!CE.pos.wrt(WORLD)
	handle None
	!ori = !!CE.ori.wrt(WORLD)
		!logdata.append('$!!CE.namtyp $!pos $!ori')
	ElseHandle Any
	Endhandle
	ROTA THR E $!this.eastdist.val N $!this.northdist.val U $!this.updist.val WRT/* ABOU $!this.zdir.val by $!this.roteval.val
	!logdata.append('Rotated $!!CE.namtyp')
	-- FUNC 'Rotated'
Enddo
!this.Reset()
!logdata.append('Completed...')
!log.writefile('OVERWRITE',!logdata)
-- $P Log File Generated at $!log
-- syscom 'START $!log'
if !!alert.confirm('Action completed, Press Yes to SAVEWORK') eq 'YES' then
	SAVEWORK
Endif

Endmethod


-------------------------------------------------------------------------------------------------------------------
Define Method .MoveItems()
-------------------------------------------------------------------------------------------------------------------
!this.InitCheck()

!dt = object datetime()
!date = !dt.string().replace(':','-').replace(' ','_') + '.log'
!log = Object File('%pdmsuser%\Move_$!date')
!logdata = Array()
!logdata.append('Report generated at ' + !dt.string())
!logdata.append(!this.frame1.tag)
!elems = !this.items.rtext
!logdata.append('###############################################################################################')
!logdata.appendarray(!elems)
!logdata.append('###############################################################################################')
!logdata.append('ELEMENTS MOVED BY E $!this.eastdist.val N $!this.northdist.val U $!this.updist.val WRT/*')
!tot = !elems.size()
if !!alert.confirm('Press Yes to confirm moving the $!tot Elements from current position to the mentioned ENU distances ?') eq 'NO' then
	return
Endif
Do !x values !elems
	$!x
	-- !func = Func
	-- if !func eq 'Moved' then
		-- !logdata.append('Skipped $!!CE.namtyp')
		-- skip
	-- Endif
	!pos = !!CE.pos.wrt(WORLD)
	handle None
	!ori = !!CE.ori.wrt(WORLD)
		!logdata.append('$!!CE.namtyp $!pos $!ori')
	ElseHandle Any
	Endhandle
	BY E $!this.eastdist.val WRT/*
	BY N $!this.northdist.val WRT/*
	BY U $!this.updist.val WRT/*
	!logdata.append('Moved $!!CE.namtyp')
	-- FUNC 'Moved'
Enddo
!this.Reset()
!logdata.append('Completed...')
!log.writefile('OVERWRITE',!logdata)
-- $P Log File Generated at $!log
-- syscom 'START $!log'
if !!alert.confirm('Action completed, Click Yes to SAVEWORK') eq 'YES' then
	SAVEWORK
Endif

Endmethod

-------------------------------------------------------------------------------------------------------------------
Define Method .AddElements(!mode is string)
-------------------------------------------------------------------------------------------------------------------
q var !mode
if !mode eq 'CE' then
	if ddep gt 2 then
		return error1 'Select SITE or ZONE'
	Endif
	var !elements Eval name for all ZONE MEM for CE
	!tag = 'CE : $!!CE.name : Total Elements = '
Elseif !mode eq 'Search' then
	!input = !!alert.input('Enter the Site name','/KBR-1100-0001/*')
	if !input eq '' then
		return error1 'input is blank'
	Endif
	var !elements Eval name for all ZONE MEM with matchwild(name of site,!input) and dbwrite
	Handle Any
		return error1 'Invalid exClickion entered...'
	Endhandle
	!tag = 'WBS Search Site : <$!input> : Total Elements = '
Endif
q var !elements.size()
!this.frame1.Tag = !tag + !elements.size().string()

!this.AddElementstoList(!elements)

Endmethod

-------------------------------------------------------------------------------------------------------------------
Define Method .AddElementstoList(!elements is Array)
-------------------------------------------------------------------------------------------------------------------
!ref = ce

!headings = split('NAMTYP,NAMTYP of Owner',',')
!data = Array()
!rtext = Array()
Do !ce values !elements
	CLAIM $!ce HIER
	Handle Any
		return error1 '$!!error.text'
	Endhandle
	!col = Array()
	Do !h values !headings
		var !res $!h of $!ce
		!col.append(!res)
	Enddo
	!data.append(!col)
	!rtext.append('$!ce')
Enddo
!this.items.setRows(!data)
!this.items.rtext = !rtext
$!ref

Endmethod


