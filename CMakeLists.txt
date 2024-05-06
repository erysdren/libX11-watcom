cmake_minimum_required(VERSION 3.23)
project(X11 VERSION 1.8.3 LANGUAGES C)

set(LIBX11_SOURCES
	AllCells.c AllowEv.c AllPlanes.c AutoRep.c Backgnd.c BdrWidth.c Bell.c
	Border.c ChAccCon.c ChActPGb.c ChClMode.c ChCmap.c ChGC.c ChKeyCon.c
	ChkIfEv.c ChkMaskEv.c ChkTypEv.c ChkTypWEv.c ChkWinEv.c ChPntCon.c ChProp.c
	ChSaveSet.c ChWAttrs.c ChWindow.c CirWin.c CirWinDn.c CirWinUp.c ClDisplay.c
	ClearArea.c Clear.c ConfWind.c Context.c ConvSel.c CopyArea.c CopyCmap.c
	CopyGC.c CopyPlane.c CrBFData.c CrCmap.c CrCursor.c CrGC.c CrGlCur.c
	CrPFBData.c CrPixmap.c CrWindow.c Cursor.c DefCursor.c DelProp.c Depths.c
	DestSubs.c DestWind.c DisName.c DrArc.c DrArcs.c DrLine.c DrLines.c
	DrPoint.c DrPoints.c DrRect.c DrRects.c DrSegs.c ErrDes.c ErrHndlr.c
	evtomask.c EvToWire.c FetchName.c FillArc.c FillArcs.c FillPoly.c FillRct.c
	FillRcts.c FilterEv.c Flush.c Font.c FontInfo.c FontNames.c FreeCmap.c
	FreeCols.c FreeCurs.c FreeEData.c FreeEventData.c FreeGC.c FreePix.c
	FSSaver.c FSWrap.c GCMisc.c Geom.c GetAtomNm.c GetColor.c GetDflt.c
	GetEventData.c GetFPath.c GetFProp.c GetGCVals.c GetGeom.c GetHColor.c
	GetHints.c GetIFocus.c GetImage.c GetKCnt.c GetMoEv.c GetNrmHint.c
	GetPCnt.c GetPntMap.c GetProp.c GetRGBCMap.c GetSOwner.c GetSSaver.c
	GetStCmap.c GetTxtProp.c GetWAttrs.c GetWMCMapW.c GetWMProto.c globals.c
	GrButton.c GrKeybd.c GrKey.c GrPointer.c GrServer.c Host.c Iconify.c
	IfEvent.c imConv.c ImText16.c ImText.c ImUtil.c InitExt.c InsCmap.c
	IntAtom.c KeyBind.c KeysymStr.c KillCl.c LiHosts.c LiICmaps.c LiProps.c
	ListExt.c LoadFont.c LockDis.c locking.c LookupCol.c LowerWin.c Macros.c
	MapRaised.c MapSubs.c MapWindow.c MaskEvent.c Misc.c ModMap.c MoveWin.c
	NextEvent.c OCWrap.c OMWrap.c OpenDis.c ParseCmd.c ParseCol.c ParseGeom.c
	PeekEvent.c PeekIfEv.c Pending.c PixFormats.c PmapBgnd.c PmapBord.c
	PolyReg.c PolyTxt16.c PolyTxt.c PropAlloc.c PutBEvent.c PutImage.c Quarks.c
	QuBest.c QuColor.c QuColors.c QuCurShp.c QuExt.c QuKeybd.c QuPntr.c
	QuStipShp.c QuTextE16.c QuTextExt.c QuTileShp.c QuTree.c RaiseWin.c
	RdBitF.c RecolorC.c ReconfWin.c ReconfWM.c Region.c RegstFlt.c RepWindow.c
	RestackWs.c RotProp.c ScrResStr.c SelInput.c SendEvent.c SetBack.c
	SetClMask.c SetClOrig.c SetCRects.c SetDashes.c SetFont.c SetFore.c
	SetFPath.c SetFunc.c SetHints.c SetIFocus.c SetLocale.c SetLStyle.c
	SetNrmHint.c SetPMask.c SetPntMap.c SetRGBCMap.c SetSOwner.c SetSSaver.c
	SetState.c SetStCmap.c SetStip.c SetTile.c SetTSOrig.c SetTxtProp.c
	SetWMCMapW.c SetWMProto.c StBytes.c StColor.c StColors.c StName.c StNColor.c
	StrKeysym.c StrToText.c Sync.c Synchro.c Text16.c Text.c TextExt16.c
	TextExt.c TextToStr.c TrCoords.c UndefCurs.c UngrabBut.c UngrabKbd.c
	UngrabKey.c UngrabPtr.c UngrabSvr.c UninsCmap.c UnldFont.c UnmapSubs.c
	UnmapWin.c VisUtil.c WarpPtr.c Window.c WinEvent.c Withdraw.c WMGeom.c
	WMProps.c WrBitF.c xcb_disp.c xcb_io.c XlibAsync.c XlibInt.c Xrm.c
)

list(TRANSFORM LIBX11_SOURCES PREPEND ${CMAKE_CURRENT_SOURCE_DIR}/src/)

# makekeys
add_executable(makekeys ${CMAKE_CURRENT_SOURCE_DIR}/src/util/makekeys.c)

# horrible hack
set(KEYSYMDEFS keysymdef.h XF86keysym.h Sunkeysym.h DECkeysym.h HPkeysym.h)
list(TRANSFORM KEYSYMDEFS PREPEND /usr/include/X11/)
add_custom_command(
	OUTPUT ${CMAKE_SOURCE_DIR}/src/ks_tables.h
	COMMAND ${CMAKE_BINARY_DIR}/makekeys ${KEYSYMDEFS} > ${CMAKE_SOURCE_DIR}/src/ks_tables.h
)
add_custom_target(ks_tables ALL DEPENDS ${CMAKE_SOURCE_DIR}/src/ks_tables.h)

# shared library
add_library(X11_so SHARED ${LIBX11_SOURCES})
set_target_properties(X11_so PROPERTIES OUTPUT_NAME "X11")
add_dependencies(X11_so ks_tables)
target_include_directories(X11_so PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_include_directories(X11_so PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include/X11)
target_include_directories(X11_so PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/xcms)
target_include_directories(X11_so PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/xkb)
target_include_directories(X11_so PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/xlibi18n)
target_compile_definitions(X11_so PRIVATE _BSD_SOURCE X11_t TRANS_CLIENT)

# static library
add_library(X11_a STATIC ${LIBX11_SOURCES})
set_target_properties(X11_a PROPERTIES OUTPUT_NAME "X11")
add_dependencies(X11_a ks_tables)
target_include_directories(X11_a PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_include_directories(X11_a PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include/X11)
target_include_directories(X11_a PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/xcms)
target_include_directories(X11_a PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/xkb)
target_include_directories(X11_a PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/xlibi18n)
