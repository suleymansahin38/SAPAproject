# SAPAproject
#Description of codes and overview of project.









codes for arduino


               void setup() {
     Serial.begin(115200); // Start serial communication
    }

        void loop() {
     sensorValue = analogRead(analogPin); // Read analog value from the sensor
    Serial.println(sensorValue); // Send the value over serial
      delay(0.1); // Delay to control the rate of data transfer
     }

codes for matlab

    % Properties that correspond to underlying components
    classdef signal_processer_tool < matlab.ui.componentcontainer.ComponentContainer
    const int analogPin = A0; // Analog pin for audio input
    int sensorValue = 0; // Variable to store the sensor value
    properties (Access = private, Transient, NonCopyable)
        Label                  matlab.ui.control.Label
        BPMLabel               matlab.ui.control.Label
        OpenFileLabel          matlab.ui.control.Label
        ExportButton           matlab.ui.control.Button
        RecordLabel            matlab.ui.control.Label
        SaveButton             matlab.ui.control.Button
        TabGroup               matlab.ui.container.TabGroup
        HearthTab              matlab.ui.container.Tab
        LowSliderLabel         matlab.ui.control.Label
        LowSlider              matlab.ui.control.Slider
        BreathTab              matlab.ui.container.Tab
        LowSlider_2Label       matlab.ui.control.Label
        HighSliderLabel        matlab.ui.control.Label
        HighSlider             matlab.ui.control.Slider
        LowSlider_2            matlab.ui.control.Slider
        FilterTypeSwitch       matlab.ui.control.Switch
        FilterTypeSwitchLabel  matlab.ui.control.Label
        DropDown               matlab.ui.control.DropDown
        Lamp                   matlab.ui.control.Lamp
        SystemOnOff            matlab.ui.control.RockerSwitch
        Button                 matlab.ui.control.Button
        FFTGraph_2             matlab.ui.control.UIAxes
        ExportedDataGraph      matlab.ui.control.UIAxes
        FFTGraph               matlab.ui.control.UIAxes
        FilteredGraph          matlab.ui.control.UIAxes
        OrginalGraph           matlab.ui.control.UIAxes
    end

    properties (Access = public)
        arduinoObj
        bpm = '---';
        breathHigh = 0.2;
        breathLow = 0.4;
        count = 0;
        data = [];
        filterOrder = 3;
        filterType = '🫀';
        filteredData = [];
        fs = 500;
        heartLow = 0.2;
        port = 'COM3';
        storedData
        time = [];
        yMax = 2;
        yMin = -2;
    end
    
    
    methods (Access = private)
        
    % clear stored datas function
    function clearDatas(comp)

        if isprop(comp, 'Label') && isprop(comp.Label, 'Text')
            comp.Label.Text = '---';
        end
    
        comp.data = [];
        comp.time = [];
        comp.count = 0;
    end

        
        % plot function
        function result = plotOrginalData(comp)
            %lineColor: 'b','k', 'r'
            hold(comp.OrginalGraph, 'on');
            plottedGraph = plot(comp.OrginalGraph, 0, 0, 'b');
            axis(comp.OrginalGraph, [0 10 comp.yMin comp.yMax]);
            result = plottedGraph;
        end

        function result = plotFiltered(comp)
            %lineColor: 'b','k', 'r'
            hold(comp.FilteredGraph, 'on');
            plottedGraph = plot(comp.FilteredGraph, 0, 0, 'k');
            axis(comp.FilteredGraph, [0 10 comp.yMin comp.yMax]);
            result = plottedGraph;
        end

        % filter function

        function result = applyFilter(comp)
            disp(['breathLow: ', comp.breathLow]);
 
            disp(['breathHigh: ', comp.breathHigh]);
            disp(['heartLow: ', comp.heartLow]);
            if strcmp(comp.filterType,'🫀')
                [b, a] = butter(comp.filterOrder, comp.heartLow, "low");
            else
                [b, a] = butter(comp.filterOrder, [comp.breathHigh comp.breathLow], "bandpass");
            end

            filteredSignal = filter(b, a, comp.data);
            result = filteredSignal;
        end
        
        % display filtered data
        function displayFilteredSignal(comp)
            plottedGraph = comp.plotFiltered();
            
            comp.filteredData = comp.applyFilter();
            disp(comp.filteredData);

            set(plottedGraph, 'XData', comp.time, 'YData', normalize(comp.filteredData));
            if mod(comp.count, 1) == 0
                axis(comp.FilteredGraph, [comp.time(comp.count)-20 comp.time(comp.count) comp.yMin comp.yMax]);
            end
        end
        
        function displayFFTGraph(comp)
            freqPlot = plot(comp.FFTGraph, 0, 0, 'r');  % You can customize the plot appearance
            grid(comp.FFTGraph, 'on');

            % Calculate and update FFT plot
            xdft = fft(comp.data);
            xdft = xdft(1:length(comp.data)/2+1);
            DF = comp.fs/length(comp.data); % frequency increment
            freqvec = 0:DF:comp.fs/2;
            set(freqPlot, 'XData', freqvec, 'YData', abs(xdft));



            freqPlot = plot(comp.FFTGraph_2, 0, 0, 'b');  % You can customize the plot appearance
            grid(comp.FFTGraph_2, 'on');

            % Calculate and update FFT plot
            xdft = fft(comp.filteredData);
            xdft = xdft(1:length(comp.filteredData)/2+1);
            DF = comp.fs/length(comp.filteredData); % frequency increment
            freqvec = 0:DF:comp.fs/2;
            set(freqPlot, 'XData', freqvec, 'YData', abs(xdft));
        end
        
        function displayBPM(comp)
           % Ensure 'MinPeakDistance' is a scalar with value less than 10
           % Set a fixed value for MinPeakDistance
           fixedMinPeakDistance = 5;
        
           % Adjust MinPeakDistance based on the length of the data
           minPeakDistance = min(fixedMinPeakDistance, numel(comp.data) - 1);
           [~, locs] = findpeaks(comp.data, 'MinPeakHeight', 0.5, 'MinPeakDistance', minPeakDistance);
           % Calculate heart rate (BPM)
           locs_length = length(locs);
           %disp(['Time Array: ', num2str(comp.time)]);
           disp(['Length of comp.time: ', num2str(length(comp.time))]);
           disp(['Number of Peaks: ', num2str(locs_length)]);


           if locs_length > 1
                % Calculate time in seconds between the first and last peak
                timeBetweenPeaks = comp.time(locs(end)) - comp.time(locs(1));
                            
                % Calculate heart rate
                beats = 60 * (locs_length - 1) / timeBetweenPeaks;
                            
                % Update the BPM in the UI
                comp.Label.Text = num2str(round(beats));
           end 
        end

        function startDisplay2Graphs(comp)

            warning('off','all');
            if ishandle(comp.OrginalGraph)
               cla(comp.OrginalGraph);
            end
            if ishandle(comp.FilteredGraph)
               cla(comp.FilteredGraph);
            end
            comp.Label.Text = '---';
            comp.data = [];
            comp.time = [];
            comp.count = 0;
            
            plottedGraph = comp.plotOrginalData();

            tic
            while strcmp(comp.SystemOnOff.Value, 'On')
                signal = readVoltage(comp.arduinoObj, 'A0');
                comp.count = comp.count + 1;
                comp.time(comp.count) = toc;
                comp.data(comp.count) = signal(1);
                set(plottedGraph, 'XData', comp.time, 'YData', normalize(comp.data));
                if mod(comp.count, 1) == 0
                    axis(comp.OrginalGraph, [comp.time(comp.count)-20 comp.time(comp.count) comp.yMin comp.yMax]);
                end

                if length(comp.time) > 10
                    comp.displayFilteredSignal();
                    comp.displayFFTGraph();
                    comp.displayBPM();
                end
            end
        end


    end
    

    % Callbacks that handle component events
    methods (Access = private)

        % Value changed function: FilterTypeSwitch
        function FilterTypeSwitchValueChanged(comp, event)
            value = comp.FilterTypeSwitch.Value;
            if strcmp(value,'🫀')
                comp.TabGroup.SelectedTab = comp.HearthTab;
            else
                comp.TabGroup.SelectedTab = comp.BreathTab;
            end
            comp.filterType = value;
        end

        % Value changed function: DropDown
        function DropDownValueChanged(comp, event)
            value = comp.DropDown.Value;
            try
                % Check if an Arduino object already exists
                %if ~isempty(comp.arduinoObj) && isvalid(comp.arduinoObj)
                %    % If so, delete the existing object
                %    delete(comp.arduinoObj);
                %end
                comp.port = value;
                
                % Create a new Arduino object
                comp.arduinoObj = arduino(comp.port, 'Uno');
                comp.Lamp.Color = [0, 1, 1];  % Set color to cyan for successful connection
            catch
                comp.Lamp.Color = [1, 0, 0];  % Set color to red for connection error
                disp('Arduino connection failed');
            end
        end

        % Value changed function: SystemOnOff
        function SystemOnOffValueChanged(comp, event)
            value = comp.SystemOnOff.Value;
            if strcmp(value, 'On')
                comp.Lamp.Color = [0.39,0.83,0.07];
                comp.startDisplay2Graphs();
            else 
                comp.Lamp.Color = [0.65,0.65,0.65];
            end
        end

        % Button pushed function: SaveButton
        function SaveButtonPushed(comp, event)
                [filename, pathname] = uiputfile({'*.mat', 'MAT Files (*.mat)'; '*.txt', 'Text Files (*.txt)'}, 'Save Signal Data');
                % Check if the user clicked Cancel
                if isequal(filename, 0) || isequal(pathname, 0)
                    disp('User canceled the operation.');
                else
                    % Construct the full file path
                    file_path = fullfile(pathname, filename);
                    
                    % Save the data (replace filteredSignal with your actual variable)
                     myData.data = comp.filteredData;
                     myData.time = comp.time;
                     save(file_path, 'myData');
                
                    disp(['Data saved to: ' file_path]);
                end
        end

        % Button pushed function: ExportButton
        function ExportButtonPushed(comp, event)
                [filename, pathname] = uigetfile({'*.mat', 'MAT Files (*.mat)'; '*.txt', 'Text Files (*.txt)'}, 'Select Data File to Export');
        
                % Check if the user clicked Cancel
                if isequal(filename, 0) || isequal(pathname, 0)
                    disp('User canceled the operation.');
                else
                    % Construct the full file path
                    file_path = fullfile(pathname, filename);
                    
                    % Load the saved data
                    loadedData = load(file_path);
                    
                    % Extract data and time from the loaded structure
                    exportedData = loadedData.myData;
                    
                    % Perform export operation using exportedData.data and exportedData.time
                    
                    % For example, you can display the loaded data in the MATLAB Command Window
                    disp('Loaded Data:');
                    disp(['Data: ' num2str(exportedData.data)]);
                    disp(['Time: ' num2str(exportedData.time)]);
                    
                    % Add your export logic here using exportedData.data and exportedData.time
                    % For instance, you might want to update some UI components or export to another format.
                    % ...
                    % Plot the loaded data in comp.ExportedDataGraph
                    plot(comp.ExportedDataGraph, exportedData.time, exportedData.data);
                    
                    % Customize the plot if needed
                    title(comp.ExportedDataGraph, 'Loaded Data');
                    xlabel(comp.ExportedDataGraph, 'Time');
                    ylabel(comp.ExportedDataGraph, 'Amplitude');
                end
        end

        % Value changed function: LowSlider
        function LowSliderValueChanged(comp, event)
            value = comp.LowSlider.Value;
            comp.heartLow = value;
        end

        % Value changed function: LowSlider_2
        function LowSlider_2ValueChanged(comp, event)
            value = comp.LowSlider_2.Value;
            comp.breathLow = value;
        end

        % Value changed function: HighSlider
        function HighSliderValueChanged(comp, event)
            value = comp.HighSlider.Value;
            comp.breathHigh = value;
        end
    end

    methods (Access = protected)
        
        % Code that executes when the value of a public property is changed
        function update(comp)
            % Use this function to update the underlying components
            
        end

        % Create the underlying components
        function setup(comp)

            comp.Position = [1 1 790 712];
            comp.BackgroundColor = [0.94 0.94 0.94];

            % Create OrginalGraph
            comp.OrginalGraph = uiaxes(comp);
            title(comp.OrginalGraph, 'Real-time Audio Signal Plot')
            xlabel(comp.OrginalGraph, 'Time (s)')
            ylabel(comp.OrginalGraph, 'Amplitude (V)')
            zlabel(comp.OrginalGraph, 'Z')
            comp.OrginalGraph.XGrid = 'on';
            comp.OrginalGraph.YGrid = 'on';
            comp.OrginalGraph.Position = [258 543 533 164];

            % Create FilteredGraph
            comp.FilteredGraph = uiaxes(comp);
            title(comp.FilteredGraph, 'Filtered Audio Signal Plot')
            xlabel(comp.FilteredGraph, 'Time (s)')
            ylabel(comp.FilteredGraph, 'Amplitude (V)')
            zlabel(comp.FilteredGraph, 'Z')
            comp.FilteredGraph.XGrid = 'on';
            comp.FilteredGraph.YGrid = 'on';
            comp.FilteredGraph.Position = [258 359 533 171];

            % Create FFTGraph
            comp.FFTGraph = uiaxes(comp);
            title(comp.FFTGraph, 'FFT of Orginal Signal')
            xlabel(comp.FFTGraph, 'Frequency (Hz)')
            ylabel(comp.FFTGraph, 'Amplitude')
            zlabel(comp.FFTGraph, 'Z')
            comp.FFTGraph.XGrid = 'on';
            comp.FFTGraph.YGrid = 'on';
            comp.FFTGraph.Position = [258 180 257 164];

            % Create ExportedDataGraph
            comp.ExportedDataGraph = uiaxes(comp);
            title(comp.ExportedDataGraph, 'Exported Data')
            xlabel(comp.ExportedDataGraph, 'Time (s)')
            ylabel(comp.ExportedDataGraph, 'Amplitude (V)')
            zlabel(comp.ExportedDataGraph, 'Z')
            comp.ExportedDataGraph.XGrid = 'on';
            comp.ExportedDataGraph.YGrid = 'on';
            comp.ExportedDataGraph.Position = [258 9 533 157];

            % Create FFTGraph_2
            comp.FFTGraph_2 = uiaxes(comp);
            title(comp.FFTGraph_2, 'FFT of Filtered Signal')
            xlabel(comp.FFTGraph_2, 'Frequency (Hz)')
            ylabel(comp.FFTGraph_2, 'Amplitude')
            zlabel(comp.FFTGraph_2, 'Z')
            comp.FFTGraph_2.YGrid = 'on';
            comp.FFTGraph_2.Position = [537 180 254 164];

            % Create Button
            comp.Button = uibutton(comp, 'push');
            comp.Button.Enable = 'off';
            comp.Button.Position = [34 94 195 96];
            comp.Button.Text = '';

            % Create SystemOnOff
            comp.SystemOnOff = uiswitch(comp, 'rocker');
            comp.SystemOnOff.ValueChangedFcn = matlab.apps.createCallbackFcn(comp, @SystemOnOffValueChanged, true);
            comp.SystemOnOff.FontWeight = 'bold';
            comp.SystemOnOff.Position = [29 608 32 73];

            % Create Lamp
            comp.Lamp = uilamp(comp);
            comp.Lamp.Position = [133 647 34 34];
            comp.Lamp.Color = [0.651 0.651 0.651];

            % Create DropDown
            comp.DropDown = uidropdown(comp);
            comp.DropDown.Items = {'COM3', 'COM4', 'COM5', 'COM6', 'COM7', 'COM8', 'COM9', 'COM10'};
            comp.DropDown.ValueChangedFcn = matlab.apps.createCallbackFcn(comp, @DropDownValueChanged, true);
            comp.DropDown.FontSize = 15;
            comp.DropDown.BackgroundColor = [1 1 1];
            comp.DropDown.Position = [96 608 115 28];
            comp.DropDown.Value = 'COM3';

            % Create FilterTypeSwitchLabel
            comp.FilterTypeSwitchLabel = uilabel(comp);
            comp.FilterTypeSwitchLabel.HorizontalAlignment = 'center';
            comp.FilterTypeSwitchLabel.FontSize = 16;
            comp.FilterTypeSwitchLabel.FontWeight = 'bold';
            comp.FilterTypeSwitchLabel.Position = [109 493 85 22];
            comp.FilterTypeSwitchLabel.Text = 'Filter Type';

            % Create FilterTypeSwitch
            comp.FilterTypeSwitch = uiswitch(comp, 'slider');
            comp.FilterTypeSwitch.Items = {'🫀', '😮‍💨'};
            comp.FilterTypeSwitch.ValueChangedFcn = matlab.apps.createCallbackFcn(comp, @FilterTypeSwitchValueChanged, true);
            comp.FilterTypeSwitch.FontSize = 40;
            comp.FilterTypeSwitch.FontColor = [0.149 0.149 0.149];
            comp.FilterTypeSwitch.Position = [124 529 52 23];
            comp.FilterTypeSwitch.Value = '🫀';

            % Create TabGroup
            comp.TabGroup = uitabgroup(comp);
            comp.TabGroup.Position = [29 287 221 174];

            % Create HearthTab
            comp.HearthTab = uitab(comp.TabGroup);
            comp.HearthTab.Title = 'Hearth';

            % Create LowSlider
            comp.LowSlider = uislider(comp.HearthTab);
            comp.LowSlider.Limits = [0.1 0.99];
            comp.LowSlider.MajorTicks = [0.1 0.2 0.35 0.5 0.65 0.8 0.99];
            comp.LowSlider.ValueChangedFcn = matlab.apps.createCallbackFcn(comp, @LowSliderValueChanged, true);
            comp.LowSlider.Position = [17 95 183 3];
            comp.LowSlider.Value = 0.2;

            % Create LowSliderLabel
            comp.LowSliderLabel = uilabel(comp.HearthTab);
            comp.LowSliderLabel.HorizontalAlignment = 'center';
            comp.LowSliderLabel.FontSize = 16;
            comp.LowSliderLabel.Position = [1 115 52 22];
            comp.LowSliderLabel.Text = 'Low:';

            % Create BreathTab
            comp.BreathTab = uitab(comp.TabGroup);
            comp.BreathTab.Title = 'Breath';

            % Create LowSlider_2
            comp.LowSlider_2 = uislider(comp.BreathTab);
            comp.LowSlider_2.Limits = [0.1 0.99];
            comp.LowSlider_2.MajorTicks = [0.1 0.2 0.35 0.5 0.65 0.8 0.99];
            comp.LowSlider_2.ValueChangedFcn = matlab.apps.createCallbackFcn(comp, @LowSlider_2ValueChanged, true);
            comp.LowSlider_2.Position = [15 116 185 3];
            comp.LowSlider_2.Value = 0.4;

            % Create HighSlider
            comp.HighSlider = uislider(comp.BreathTab);
            comp.HighSlider.Limits = [0.1 0.99];
            comp.HighSlider.MajorTicks = [0.1 0.2 0.35 0.5 0.65 0.8 0.99];
            comp.HighSlider.ValueChangedFcn = matlab.apps.createCallbackFcn(comp, @HighSliderValueChanged, true);
            comp.HighSlider.Position = [15 34 185 3];
            comp.HighSlider.Value = 0.2;

            % Create HighSliderLabel
            comp.HighSliderLabel = uilabel(comp.BreathTab);
            comp.HighSliderLabel.HorizontalAlignment = 'center';
            comp.HighSliderLabel.FontSize = 16;
            comp.HighSliderLabel.Position = [5 51 48 22];
            comp.HighSliderLabel.Text = 'High:';

            % Create LowSlider_2Label
            comp.LowSlider_2Label = uilabel(comp.BreathTab);
            comp.LowSlider_2Label.HorizontalAlignment = 'center';
            comp.LowSlider_2Label.FontSize = 16;
            comp.LowSlider_2Label.Position = [2 128 43 22];
            comp.LowSlider_2Label.Text = 'Low:';

            % Create SaveButton
            comp.SaveButton = uibutton(comp, 'push');
            comp.SaveButton.ButtonPushedFcn = matlab.apps.createCallbackFcn(comp, @SaveButtonPushed, true);
            comp.SaveButton.FontSize = 15;
            comp.SaveButton.Position = [137 243 100 27];
            comp.SaveButton.Text = 'Save';

            % Create RecordLabel
            comp.RecordLabel = uilabel(comp);
            comp.RecordLabel.FontSize = 20;
            comp.RecordLabel.FontWeight = 'bold';
            comp.RecordLabel.Position = [31 244 81 29];
            comp.RecordLabel.Text = 'Record:';

            % Create ExportButton
            comp.ExportButton = uibutton(comp, 'push');
            comp.ExportButton.ButtonPushedFcn = matlab.apps.createCallbackFcn(comp, @ExportButtonPushed, true);
            comp.ExportButton.FontSize = 15;
            comp.ExportButton.Position = [137 42 100 27];
            comp.ExportButton.Text = 'Export';

            % Create OpenFileLabel
            comp.OpenFileLabel = uilabel(comp);
            comp.OpenFileLabel.FontSize = 20;
            comp.OpenFileLabel.FontWeight = 'bold';
            comp.OpenFileLabel.Position = [31 44 104 26];
            comp.OpenFileLabel.Text = 'Open File: ';

            % Create BPMLabel
            comp.BPMLabel = uilabel(comp);
            comp.BPMLabel.FontSize = 30;
            comp.BPMLabel.FontWeight = 'bold';
            comp.BPMLabel.Position = [96 189 82 39];
            comp.BPMLabel.Text = 'BPM:';

            % Create Label
            comp.Label = uilabel(comp);
            comp.Label.HorizontalAlignment = 'center';
            comp.Label.FontSize = 30;
            comp.Label.Position = [55 116 153 53];
            comp.Label.Text = '---';
        end
    end
end
