import os
import wx
import wx.animate
from wx.lib.pubsub.core.publisher import Publisher
from PIL import Image
 

class MyFrame(wx.Frame):
    def __init__(self, parent, id, title):
        wx.Frame.__init__(self, parent, id, title, wx.DefaultPosition,
                          wx.Size(1000,1000))
        self.current_filename = ''
        self.data = {}

        self.lowerSizer = wx.BoxSizer(wx.HORIZONTAL)
        self.mainSizer = wx.BoxSizer(wx.VERTICAL)

        
        
        self.tool_bar = self.CreateToolBar()#ToolBar(self)
        #next_icon = './go-next-4.png'
        save_icon = './document-save-2.png'
        reset_icon ='./edit-delete-6.png'
        undo_icon = './edit-undo-3.png'
        open_icon = './document-open-4.png'
        opentool = self.tool_bar.AddLabelTool(wx.ID_ANY, 'open an image', 
                                              wx.Bitmap(open_icon))
        #nexttool = self.tool_bar.AddLabelTool(wx.ID_ANY, 'next image',
        #                                      wx.Bitmap(next_icon))
        resettool = self.tool_bar.AddLabelTool(wx.ID_ANY, 'clear marks',
                                              wx.Bitmap(reset_icon))
        undotool = self.tool_bar.AddLabelTool(wx.ID_ANY, 'undo mark', 
                                              wx.Bitmap(undo_icon))
        savetool = self.tool_bar.AddLabelTool(wx.ID_ANY, 'Quit',
                                              wx.Bitmap(save_icon))
        self.tool_bar.Realize()

        self.Bind(wx.EVT_TOOL, self.OnReset, resettool)
        self.Bind(wx.EVT_TOOL, self.OnBrowse, opentool)
        #self.Bind(wx.EVT_TOOL, self.OnNext, nexttool)
        self.Bind(wx.EVT_TOOL, self.OnUndo, undotool)
        self.Bind(wx.EVT_TOOL, self.OnSaveRequest, savetool)

        self.img_panel = ImgPanel(self)
        self.ctrl_panel = CtrlPanel(self)

#        self.mainSizer.Add(self.tool_bar, 0, wx.ALL, 5)
        self.lowerSizer.Add(self.img_panel, 0, wx.ALL,5)
        self.lowerSizer.Add(self.ctrl_panel, 1, wx.EXPAND, 5)
        self.mainSizer.Add(self.lowerSizer, 0, wx.ALL, 5)
        self.SetSize((1000,1000))
        self.SetSizer(self.mainSizer)
        self.mainSizer.Fit(self)

        Publisher().subscribe(self.OnImgLeft, ("img.leftclick"))
        Publisher().subscribe(self.OnImgRight, ("img.rightclick"))

        self.click_history = []

###################
# Event handlers

    def OnUndo(self, event):
        try:
            click_btn = self.click_history.pop(-1)
            if click_btn == 'L':
                self.img_panel.livePos.pop(-1)
                self.img_panel.RePaint()
                self.ctrl_panel.livecount -=1
                count = self.ctrl_panel.livecount
                self.ctrl_panel.liveCtrl.SetValue(str(count))
            if click_btn == 'R':
                self.img_panel.deadPos.pop(-1)
                self.img_panel.RePaint()
                self.ctrl_panel.deadcount -=1
                count = self.ctrl_panel.deadcount
                self.ctrl_panel.deadCtrl.SetValue(str(count))
        except IndexError:
            pass

    def OnNext(self,event):
        if self.current_filename == '':
            return
        elif self.current_filename.endswith('H12.gif'):
            return
        else:
            self.Add_data()

    def OnReset(self,event):
        self.click_history = []
        Publisher().sendMessage(("img.Reset"))

    def OnImgLeft(self,msg):
        self.click_history.append('L')

    def OnImgRight(self,msg):
        self.click_history.append('R')

    def OnBrowse(self, event):
        """ 
        Browse for file
        """
        if self.current_filename != '':
            self.Add_data()
        wildcard = "image files (*.*|*.*"
        dialog = wx.FileDialog(None, "Choose a starting file",
                               wildcard=wildcard, style=wx.OPEN)
        if dialog.ShowModal() == wx.ID_OK or True:
            filename = dialog.GetPath()
            self.current_filename = filename
            self.SetTitle('Dead or Alive: '+filename)
            Publisher().sendMessage(("img.filename"), filename)
        dialog.Destroy()

    def OnSaveRequest(self, event):
        wildcard = "CSV files (*.csv)|*.csv"
        dialog = wx.FileDialog(None, "Choose a save file",
                               wildcard=wildcard,
                               style=wx.SAVE)
        if dialog.ShowModal() == wx.ID_OK:
            filename = dialog.GetPath()
            self.SetTitle(filename)
            self.Save_data(filename)
        dialog.Destroy()

####################
# Utility functions

    def Add_data(self):            
#        curr_well = self.current_filename[:-4].split('_')[-1]
#        well_index = self.wells.index(curr_well)
#        next_well = self.wells[well_index+1]
        #new_filename = self.current_filename[:-4]
        #new_filename = new_filename.split('_')[:-1]
        #new_filename = '_'.join(new_filename) + '_' + next_well + '.gif'
        #self.current_filename = new_filename
        #self.SetTitle(new_filename)
        self.data.update({self.current_filename:(self.img_panel.livePos,\
                                     self.img_panel.deadPos)})
        #Publisher().sendMessage(("img.filename"), new_filename)

    def Save_data(self, outfile_name):
        outfile = open(outfile_name, 'wb')
        header = 'Well\tLive_count\tDead_count\tLive_coords\tDead_coords\n'
        outfile.writelines(header)
        self.Add_data()
        print self.data
        for item in self.data:
            marks = self.data[item]
            data_line = [item, str(len(marks[0])), str(len(marks[1]))]#,
                         #str(marks[0]), str(marks[1])]
            data_line = '\t'.join(data_line) + '\n'
            outfile.writelines(data_line)
        outfile.close()
        return True


################################################################################


class ToolBar(wx.Panel):
    def __init__(self, parent, *args, **kwargs):
        wx.Panel.__init__(self, parent=parent)

        self.browseBtn = wx.Button(self,id=wx.ID_OPEN, label='')
        self.browseBtn.Bind(wx.EVT_BUTTON, self.onBrowse)
        self.photoTxt = wx.TextCtrl(self, size=(200,-1))
         
        self.resetBtn = wx.Button(self, label='Reset')
        self.resetBtn.Bind(wx.EVT_BUTTON, self.onReset)

        self.undoBtn = wx.Button(self, label='Undo')

        self.panel_sizer = wx.BoxSizer(wx.HORIZONTAL)
        self.panel_sizer.Add(self.browseBtn, 0, wx.ALL, 5)        
        self.panel_sizer.Add(self.photoTxt, 0, wx.ALL, 5)
        self.panel_sizer.Add(self.resetBtn, 0, wx.ALL, 5)
        self.panel_sizer.Add(self.undoBtn, 0, wx.ALL, 5)
        self.SetSizer(self.panel_sizer)
    
        self.Layout()



################################################################################


class ImgPanel(wx.Panel):
    def __init__(self, parent, *args, **kwargs):
        wx.Panel.__init__(self, parent=parent)
        

        self.timer=wx.Timer(self)
        self.timer.Start(500)
        self.display_size = wx.GetDisplaySize()

        img = wx.EmptyImage(250,250)
        self.img_list = [img, img, img]
        self.img_index = 0

        self.imageCtrl1 = wx.StaticBitmap(self, wx.ID_ANY, 
                                         wx.BitmapFromImage(img))

        self.panel_sizer = wx.BoxSizer(wx.HORIZONTAL)
        self.panel_sizer.Add(self.imageCtrl1, 1, wx.ALL, 5)
        self.SetSizer(self.panel_sizer)
        self.Layout()
        
        self.Bind(wx.EVT_TIMER, self.OnTimer)

        Publisher().subscribe(self.OnView, ("img.filename"))
        Publisher().subscribe(self.OnReset, ("img.Reset"))

        self.pos = ()
        self.livePos = []
        self.deadPos = []

    def OnReset(self, msg):
        self.livePos = []
        self.deadPos = []
        self.Refresh()

    def OnTimer(self, event):
        self.RePaint()

    def RePaint(self):
        dc = wx.ClientDC(self.imageCtrl1)
        frameImage=self.img_list[self.img_index]
        bmp = frameImage.ConvertToBitmap()
        dc.DrawBitmap(bmp,5,5)
        dc.SetPen(wx.Pen('FOREST GREEN',5))
        for pos in self.livePos:
            dc.DrawCircle(pos.x+4, pos.y+4, 2)
        dc.SetPen(wx.Pen('RED',5))
        for pos in self.deadPos:
            dc.DrawCircle(pos.x+4, pos.y+4, 2)
        if self.img_index == len(self.img_list)-1:
            self.img_index=0
        else:
            self.img_index+=1
        self.Show()

    def OnMove(self, event):
        pos = event.GetPosition()
        self.pos = pos
        Publisher().sendMessage(("img.position"), pos)

    def OnLeftClick(self, event):
        pos = self.pos
        self.livePos.append(pos)
        self.RePaint()
        Publisher().sendMessage(("img.leftclick"), pos)

    def OnRightClick(self, event):
        pos = self.pos
        self.deadPos.append(pos)
        self.RePaint()
        Publisher().sendMessage(("img.rightclick"), pos)

    def OnView(self,msg):
        filepath = msg.data
        self.img_list = []
        frame = Image.open(filepath)
        frame_size = frame.size
        w_ratio = frame_size[0] / (7.0 * self.display_size[0] / 8)
        h_ratio = frame_size[1] / (7.0 *self.display_size[1] / 8)
        if w_ratio > h_ratio and w_ratio > 1:
            new_w = int(frame_size[0] / w_ratio)
            new_h = int(frame_size[1] / w_ratio)
            frame = frame.resize((new_w, new_h), Image.ANTIALIAS)
        elif  h_ratio > w_ratio and h_ratio > 1:
            new_w = int(frame_size[0] / h_ratio)
            new_h = int(frame_size[1] / h_ratio)
            frame = frame.resize((new_w, new_h), Image.ANTIALIAS)
        else: pass


        #x = 3.0 / 0


        nframes = 0
        self.img_index=0
        while frame:
            frameWX = wx.EmptyImage(frame.size[0], frame.size[1] )
            frameWX.SetData( frame.convert( 'RGB' ).tostring() )
            self.img_list.append(frameWX)
            nframes += 1
            try:
                frame.seek(nframes)
            except EOFError:
                break


        self.imageCtrl1.Bind(wx.EVT_MOTION,  self.OnMove)
        self.imageCtrl1.Bind(wx.EVT_LEFT_DOWN,  self.OnLeftClick)
        self.imageCtrl1.Bind(wx.EVT_RIGHT_DOWN,  self.OnRightClick)

        self.imageCtrl1.SetCursor(wx.StockCursor(wx.CURSOR_CROSS))


        self.panel_sizer.Remove(0)
        img = wx.EmptyImage(frame.size[0], frame.size[1])
        self.imageCtrl1 = wx.StaticBitmap(self, wx.ID_ANY, 
                                         wx.BitmapFromImage(img))

        self.panel_sizer.Clear(deleteWindows=True)
        self.panel_sizer = wx.BoxSizer(wx.HORIZONTAL)
        self.panel_sizer.Add(self.imageCtrl1, 0, wx.ALL, 5)
        self.SetSizer(self.panel_sizer)

        self.imageCtrl1.Bind(wx.EVT_MOTION,  self.OnMove)
        self.imageCtrl1.Bind(wx.EVT_LEFT_DOWN,  self.OnLeftClick)
        self.imageCtrl1.Bind(wx.EVT_RIGHT_DOWN,  self.OnRightClick)

        self.imageCtrl1.SetCursor(wx.StockCursor(wx.CURSOR_CROSS))

        self.Layout()
        
        self.Refresh()
        Publisher().sendMessage(("img.Reset"), True)
        #self.Refresh()

################################################################################


class CtrlPanel(wx.Panel):
    def __init__(self, parent, *args, **kwargs):
        wx.Panel.__init__(self, parent=parent)
        self.posCtrl = wx.TextCtrl(self, -1, "")
        self.label1 = wx.StaticText(self, label="Cursor position:\t\t\t ")
        self.liveCtrl = wx.TextCtrl(self, -1, "")
        self.label2 = wx.StaticText(self, label="Live worm counts:\t\t")
        self.deadCtrl = wx.TextCtrl(self, -1, "")
        self.label3 = wx.StaticText(self, label="Dead worm counts:\t\t")
        
        self.panel_sizer = wx.BoxSizer(wx.VERTICAL)

        self.data1_sizer = wx.BoxSizer(wx.HORIZONTAL)
        self.data1_sizer.Add(self.label1, 0, wx.ALL, 5)
        self.data1_sizer.Add(self.posCtrl, 0, wx.ALL, 5)
        self.panel_sizer.Add(self.data1_sizer, 0, wx.EXPAND, 5)        

        self.data2_sizer = wx.BoxSizer(wx.HORIZONTAL)
        self.data2_sizer.Add(self.label2, 0, wx.ALL, 5)
        self.data2_sizer.Add(self.liveCtrl, 0, wx.ALL, 5)
        self.panel_sizer.Add(self.data2_sizer, 0, wx.EXPAND, 5)        

        self.data3_sizer = wx.BoxSizer(wx.HORIZONTAL)
        self.data3_sizer.Add(self.label3, 0, wx.ALL, 5)
        self.data3_sizer.Add(self.deadCtrl, 0, wx.ALL, 5)
        self.panel_sizer.Add(self.data3_sizer, 0, wx.EXPAND, 5)        


        self.SetSizer(self.panel_sizer)

        self.livecount = 0
        self.deadcount = 0

        Publisher().subscribe(self.OnMove, ("img.position"))
        Publisher().subscribe(self.OnImgLeft, ("img.leftclick"))
        Publisher().subscribe(self.OnImgRight, ("img.rightclick"))
        Publisher().subscribe(self.OnImgNew, ("img.Reset"))

    def OnMove(self,msg):
        pos=msg.data
        self.posCtrl.SetValue("%s, %s" % (pos.x, pos.y))

    def OnImgLeft(self,msg):
        self.livecount += 1
        self.liveCtrl.SetValue(str(self.livecount))

    def OnImgRight(self,msg):
        self.deadcount += 1
        self.deadCtrl.SetValue(str(self.deadcount))

    def OnImgNew(self, msg):
        self.livecount = 0
        self.liveCtrl.SetValue(str(self.livecount))
        self.deadcount = 0
        self.deadCtrl.SetValue(str(self.deadcount))



################################################################################


class MyApp(wx.App):        
    def OnInit(self, filename=None):
        frame = MyFrame(None, -1, 'Dead or Alive')
        frame.Show(True)
        frame.Centre()
        return True


 
if __name__ == '__main__':
    app = MyApp()
    app.MainLoop()
