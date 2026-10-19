mkfile_path := $(abspath $(lastword $(MAKEFILE_LIST)))
current_dir := $(notdir $(patsubst %/,%,$(dir $(mkfile_path))))

target = $(current_dir)
pcb = $(target).kicad_pcb
sch = $(target).kicad_sch

gerber_dir = gerber
export_dir = $(gerber_dir)/export
zip = zip -j

.PHONY: all gerbers export drc erc 3d clean pristine

all: $(target).zip

gerbers: $(pcb)
	@mkdir -p $(gerber_dir)
	kicad-cli pcb export gerbers \
		--layers F.Cu,B.Cu,F.Mask,B.Mask,F.SilkS,B.SilkS,Edge.Cuts \
		--subtract-soldermask \
		--output $(gerber_dir)/ $(pcb)
	kicad-cli pcb export drill \
		--output $(gerber_dir)/ $(pcb)

$(target).zip: gerbers
	@mkdir -p $(export_dir)
	cp $(gerber_dir)/$(target)-B_Cu.gbl $(export_dir)/$(target).gbl
	cp $(gerber_dir)/$(target)-B_Mask.gbs $(export_dir)/$(target).gbs
	cp $(gerber_dir)/$(target)-B_Silkscreen.gbo $(export_dir)/$(target).gbo
	cp $(gerber_dir)/$(target)-Edge_Cuts.gm1 $(export_dir)/$(target).gko
	cp $(gerber_dir)/$(target)-F_Cu.gtl $(export_dir)/$(target).gtl
	cp $(gerber_dir)/$(target)-F_Mask.gts $(export_dir)/$(target).gts
	cp $(gerber_dir)/$(target)-F_Silkscreen.gto $(export_dir)/$(target).gto
	cp $(gerber_dir)/$(target).drl $(export_dir)/$(target).drl
	@rm -f $@
	$(zip) $@ $(export_dir)/*
	@rm -rf $(export_dir)

export: $(target).zip

drc: $(pcb)
	kicad-cli pcb drc $(pcb)

erc: $(sch)
	kicad-cli sch erc $(sch)

3d: $(pcb)
	-kicad-cli pcb export step --output $(target).step $(pcb)
	-kicad-cli pcb export glb --output $(target).glb $(pcb)

clean:
	-rm -f $(target).zip
	-rm -rf $(export_dir)
	-rm -f $(gerber_dir)/$(target)-* $(gerber_dir)/$(target).drl $(gerber_dir)/$(target)-job.gbrjob
	-find . -type f -name "*~" -exec rm {} \;
	-find . -type f -name ".*.sw?" -exec rm {} \;

pristine: clean
	-rm -f *.bak *.kicad_pcb-bak *.rpt *.step *.glb
	-rm -rf $(export_dir)

# vi: set sts=4 sw=4 ts=4:
