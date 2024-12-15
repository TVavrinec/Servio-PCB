
all: out/servio/gerbers.zip out/servio_dbg/gerbers.zip

servio.kicad_pcb: mainBoard.kicad_pcb
	kikit separate --source 'rectangle; tlx: 90mm; tly: 55mm; brx: 115mm; bry: 90mm' $^ $@

servio_dbg.kicad_pcb: mainBoard.kicad_pcb
	kikit separate --source 'rectangle; tlx: 120mm; tly: 45mm; brx: 215mm; bry: 110mm' $^ $@

out/servio/gerbers.zip: servio.kicad_pcb
	kikit fab jlcpcb $^ out/servio/ --assembly --schematic ./mainBoard.kicad_sch

out/servio_dbg/gerbers.zip: servio_dbg.kicad_pcb
	kikit fab jlcpcb $^ out/servio_dbg/ --assembly --schematic ./mainBoard.kicad_sch
