/*
	Copy this in bottom of cl_hooks.lua if you don't have any idea where to put this
	code.

	or Just make your own derma file and include this.
*/

surface.CreateFont("nut_MenuButtonFont", {
	font = mainFont,
	size = ScreenScale(10),
	weight = 1000,
	antialias = true
})

surface.CreateFont("nut_MenuButtonFont2", {
	font = mainFont,
	size = ScreenScale(11),
	weight = 1000,
	antialias = true
})

	local gradient = surface.GetTextureID("gui/gradient_up")
	local gradient2 = surface.GetTextureID("gui/gradient_down")
	local gradient3 = surface.GetTextureID("gui/center_gradient")
	local gradient4 = surface.GetTextureID("gui/gradient")
	
	local PANEL = {}

	function PANEL:Init()
		surface.SetFont("nut_MenuButtonFont")
		local _, height = surface.GetTextSize("W")

		self:SetTall(height + 16)
		self:DockMargin(0, 0, 0, 8)
		self:Dock(TOP)
		self:SetDrawBackground(false)
		self:SetContentAlignment( 5 )
		self:SetFont("nut_MenuButtonFont2")
		self:SetTextColor(Color(240, 240, 240))
		self:SetExpensiveShadow(1, color_black)
		self.colorApproach = 0
		self.colore = 0
	end
	function PANEL:OnHover() end
	function PANEL:OnUnHover() end
	function PANEL:OnCursorEntered()
		surface.PlaySound("weapons/smg1/switch_single.wav")
		self.colore = 1
		self:OnHover()
	end
	function PANEL:OnCursorExited()
		self.colore = 0
		self:OnUnHover()
	end
	function PANEL:Paint(w, h)
		self.colorApproach = math.Approach(self.colorApproach, self.colore, FrameTime() * 4)
		local blink = 0
		if (self.alphaApproach == HOVER_COLOR) then
			blink = math.abs(math.sin(RealTime() * 2))/2
		end			
		surface.SetDrawColor(100*self.colorApproach, 100*self.colorApproach, 0, 120)
		surface.SetTexture(gradient3)
		surface.DrawTexturedRect(0, h*.1, w, h*.8)
	end
	vgui.Register("nut_MainMenuButton", PANEL, "DButton")



	local PANEL = {}
	
	local line = .1

	function PANEL:Init()
		self:SetSize(ScrW(), ScrH())
		self:SetDrawBackground(false)
		self:DockPadding(ScrW()*line/2,ScrH()*line*1.5,ScrW()*line/2,ScrH()*line*1.5)
							self:MakePopup()

		self.title = self:Add("DLabel")
		self.title:Dock(TOP)
		self.title:SetText(SCHEMA.name)
		self.title:SetFont("nut_TitleFont")
		self.title:SizeToContents()
		self.title:SetContentAlignment( 5 )
		self.title:SetTextColor(color_white)
		self.title:SetExpensiveShadow(3, color_black)
		self.title.Paint = function(panel, w, h)
			surface.SetDrawColor(0, 0, 0, 125)
			surface.SetTexture(gradient3)
			surface.DrawTexturedRect(w * 0.25, 0, w * 0.5, h)
		end
		
		self.subTitle = self:Add("DLabel")
		self.subTitle:Dock(TOP)
		self.subTitle:SetText(SCHEMA.desc)
		self.subTitle:SetFont("nut_SubTitleFont")
		self.subTitle:SizeToContents()
		self.subTitle:SetContentAlignment( 5 )
		self.subTitle:SetTextColor(color_white)
		self.subTitle:SetExpensiveShadow(2, color_black)

		self.rightPanel = self:Add("DPanel")
		self.rightPanel:Dock(LEFT)
		self.rightPanel:DockMargin(0, 0, 0, 0)
		self.rightPanel:SetWide(ScrW() * 0.1)
		self.rightPanel.Paint = function(panel, w, h)
		end
		
		self.selectPanel = self:Add("DPanel")
		self.selectPanel:Dock(RIGHT)
		self.selectPanel:DockMargin(32, 0, 0, 0)
		self.selectPanel:SetWide(ScrW() * 0.2)
		self.selectPanel.Paint = function(panel, w, h)
		end
		
		local MODEL_ANGLE = Angle(0, 30, 0)
		local MODEL_VECTOR = Vector(0, 0, -2)
		self.model = self:Add("DModelPanel")
		self.model:SetSize( self:GetWide()/3, self:GetTall()-self:GetTall()*(line*2) )
		self.model:SetPos( self:GetWide()/3*1.7, self:GetTall()*line)
		self.model:ParentToHUD( )
	--	self.model:DockMargin(ScrW() * 0.175, ScrH() * 0.05, 0, ScrH() * 0.05)
		self.model:SetFOV(40)
		self.model.OnCursorEntered = function() end
		self.model:SetDisabled(true)
		self.model:SetCursor("none")
		local SetModel = self.model.SetModel

		self.model.SetModel = function(panel, model)
			SetModel(panel, model)

			local entity = panel.Entity
			local sequence = entity:LookupSequence("idle")

			if (sequence <= 0) then
				sequence = entity:LookupSequence("idle_subtle")
			end

			if (sequence <= 0) then
				sequence = entity:LookupSequence("batonidle2")
			end

			if (sequence <= 0) then
				sequence = entity:LookupSequence("idle_unarmed")
			end

			if (sequence <= 0) then
				sequence = entity:LookupSequence("idle01")
			end

			if (sequence > 0) then
				entity:ResetSequence(sequence)
			end
		end	
	
		self.model.LayoutEntity = function(panel, entity)
			if (!IsValid(self)) then
				panel:Remove()
			
				return
			end
			
			local xRatio = gui.MouseX() / ScrW()
			local yRatio = gui.MouseY() / ScrH()

			entity:SetPoseParameter("head_pitch", yRatio*60 - 30)
			entity:SetPoseParameter("head_yaw", (xRatio - 0.75)*60)
			entity:SetIK( false )
			entity:SetAngles(MODEL_ANGLE)
			entity:SetPos(MODEL_VECTOR)

			panel:RunAnimation()
		end
	
		
		local function addMainButton(variable, text)
			local button = self.rightPanel:Add("nut_MainMenuButton")
			button:Dock(BOTTOM)
			button:DockMargin(0, 0, 0, 0)
			button:SetText(text)
			button:SetTall(ScrH()*.05)
	
			self[variable] = button
		end
	
		addMainButton("leave", nut.loaded and nut.lang.Get("return") or nut.lang.Get("leave"))
		self.leave:SetTextColor( Color( 240, 160, 160 ) )
		self.leave.DoClick = function()
			if (LocalPlayer().character) then
				if (IsValid(nut.gui.charCreate)) then
					return false
				end

				self:FadeOutMusic()

				if (IsValid(self.model)) then
					self.model:Remove()
				end

				self:Remove()
			else
				RunConsoleCommand("disconnect")
			end
		end
		
		if (LocalPlayer().characters and table.Count(LocalPlayer().characters) > 0) then -- (nut.faction.Count() > 0)
			addMainButton("delete", nut.lang.Get("delete"))
			self.delete.DoClick = function()
				surface.PlaySound("buttons/lightswitch2.wav")
				self.selectPanel:Clear()
				
				for _, v in pairs(LocalPlayer().characters) do
					if !v.name then continue end
					local button = self.selectPanel:Add("nut_MainMenuButton")
					button:Dock(BOTTOM)
					button:DockMargin(0, 0, 0, 0)
					button:SetTextColor( Color( 255, 199, 199 ) )
					button:SetText(v.name)
					button:SetTall(ScrH()*.05)
					button.id = v.id
					button.DoClick = function()
						if (IsValid(nut.gui.charCreate)) then
							return false
						end
						
						if (!v.id) then
							return false
						end

						Derma_Query("Are you sure you want to delete this character? It can not be undone.", "Confirm", "No", nil, "Yes", function()
							button:Remove()
							button = nil

							netstream.Start("nut_CharDelete", v.id)

							for k, a in pairs(LocalPlayer().characters) do
								if (a.id == v.id) then
									LocalPlayer().characters[k] = nil
								end
							end

							surface.PlaySound("buttons/button9.wav")

							timer.Simple(0, function()
								if (IsValid(nut.gui.charMenu)) then
									nut.gui.charMenu:FadeOutMusic()
									nut.gui.charMenu:Remove()
								end
								nut.gui.charMenu = vgui.Create("nut_CharMenu")
							end)
						end)
					end
					button.OnHover = function()		
						surface.PlaySound("buttons/lightswitch2.wav")
						self.model:SetModel(v.model)
					end
					button.OnUnHover = function()
						self.model:SetModel("models/effects/teleporttrail.mdl")
					end
				end
			end
		end
		if (LocalPlayer().characters and table.Count(LocalPlayer().characters) > 0) then -- (nut.faction.Count() > 0)
			addMainButton("load", nut.lang.Get("choose"))
			self.load.DoClick = function()
				surface.PlaySound("buttons/lightswitch2.wav")
				self.selectPanel:Clear()
				
				for _, v in pairs(LocalPlayer().characters) do
					if !v.name then continue end
					local button = self.selectPanel:Add("nut_MainMenuButton")
					button:Dock(BOTTOM)
					button:DockMargin(0, 0, 0, 0)
					button:SetText(v.name)
					button:SetTall(ScrH()*.05)
					button.id = v.id
					button.DoClick = function()
						surface.PlaySound("buttons/button4.wav")	

						if (v.id) then
							netstream.Start("nut_CharChoose", v.id)
						else
							return false
						end
					end
					button.OnHover = function()		
						surface.PlaySound("buttons/lightswitch2.wav")
						self.model:SetModel(v.model)
					end
					button.OnUnHover = function()
						self.model:SetModel("models/effects/teleporttrail.mdl")
					end
				end
			end
		end
	
		if (nut.faction.Count() > 0) then -- (nut.faction.Count() > 0)
			addMainButton("create", nut.lang.Get("create"))
			self.create.DoClick = function()
				surface.PlaySound("buttons/lightswitch2.wav")
				self.selectPanel:Clear()
				
				for k, v in ipairs(nut.faction.GetAll()) do
					if (nut.faction.CanBe(LocalPlayer(), v.index)) then
						local button = self.selectPanel:Add("nut_MainMenuButton")
						button:Dock(BOTTOM)
						button:DockMargin(0, 0, 0, 0)
						button:SetText(v.name)
						button:SetTall(ScrH()*.05)
						button.DoClick = function()
							surface.PlaySound("buttons/button4.wav")	

							if (LocalPlayer().characters and table.Count(LocalPlayer().characters) >= nut.config.maxChars) then
								return false
							end
							nut.gui.charCreate = vgui.Create("nut_CharCreate")
							nut.gui.charCreate:SetupFaction(k)
						end
					end
				end
			end
		end

		if (nut.config.menuMusic) then
			if (nut.menuMusic) then
				nut.menuMusic:Stop()
				nut.menuMusic = nil
			end

			local lower = string.lower(nut.config.menuMusic)

			if (string.Left(lower, 4) == "http") then
				local function createMusic()
					if (!IsValid(nut.gui.charMenu)) then
						return
					end

					local nextAttempt = 0

					sound.PlayURL(nut.config.menuMusic, "noplay", function(music)
						if (music) then
							nut.menuMusic = music
							nut.menuMusic:Play()

							timer.Simple(0.5, function()
								if (!nut.menuMusic) then
									return
								end
								
								nut.menuMusic:SetVolume(nut.config.menuMusicVol / 100)
							end)
						elseif (nextAttempt < CurTime()) then
							nextAttempt = CurTime() + 1
							createMusic()
						end
					end)
				end

				createMusic()
			else
				nut.menuMusic = CreateSound(LocalPlayer(), nut.config.menuMusic)
				nut.menuMusic:Play()
				nut.menuMusic:ChangeVolume(nut.config.menuMusicVol / 100, 0)
			end
		end

		nut.loaded = true
	end

	function PANEL:FadeOutMusic()
		if (!nut.menuMusic) then
			return
		end

		if (nut.menuMusic.SetVolume) then
			local start = CurTime()
			local finish = CurTime() + nut.config.menuMusicFade

			if (timer.Exists("nut_FadeMenuMusic")) then
				timer.Remove("nut_FadeMenuMusic")

				if (nut.menuMusic) then
					nut.menuMusic:Stop()
					nut.menuMusic = nil
				end
			end

			timer.Create("nut_FadeMenuMusic", 0, 0, function()
				local fraction = (1 - math.TimeFraction(start, finish, CurTime())) * nut.config.menuMusicVol

				if (nut.menuMusic) then
					nut.menuMusic:SetVolume(fraction / 100)

					if (fraction <= 0) then
						nut.menuMusic:SetVolume(0)
						nut.menuMusic:Stop()
						nut.menuMusic = nil
					end
				end
			end)
		else
			nut.menuMusic:FadeOut(nut.config.menuMusicFade)
			nut.menuMusic = nil
		end
	end

	
	function PANEL:Paint(w, h)		
		surface.SetDrawColor(0, 0, 0,255)
		surface.DrawRect(0, 0, w, h*line)
		surface.DrawRect(0, h-h*line, w, h*line)
	end
vgui.Register("nut_CharMenu", PANEL, "DPanel")
