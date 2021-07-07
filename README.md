close all;
clear;
clc;
N = input('How many files are being tested? ');

index = 1;
fLoop = 1;
while fLoop == 1
    %Entering the file name and checking if it is empty 
    fprintf('Enter Filename %d:', index);
    filename_root = input('File name (no xlsx)','s');
    if filename_root == "" 
        return; %If the file name is empty jump out the program  
    end    
    filename = strcat(filename_root,'.xlsx');
    %check to see if the given file name exists.If exists do above else try again
    if exist(filename,'file') ~= 0
        out = xlsread(filename);%Read the excel file and save it into out
        [Yn,Xn] = size(out);    %determine the number of rows and columns
        yy=out(:,1);            %selecting the first column 
        xx=out(Yn,:);           %selecting the last row 
        yi=yy;
        xi=xx;
        xx(1)=[];               %Deleting the nan from the row   
        yy(Yn)=[];              %Deleting the nan from the column 
        %Recreating the output matrix by removing the x&y-axis and only
        %including the data. Recalculating the new size.
        out=out(1:Yn-1,:);    
        out=out(:,2:Xn);
        [Yn,Xn] = size(out);
        % Checking to see if all files have the same size
        if index == 1
            Yref = Yn;
            Xref = Xn;
        elseif Yref ~= Yn || Xref ~= Xn
             disp('File size not match. Try again'); 
             continue;
        end
        %Saving each file in each dimention of A_Array provided by index 
        A_Array(index,:,:)=out;
        index = index+1;
        if index > N %Checking to see if the last file is being reached 
            fLoop = 0;
        end
     else        
            disp('File not found. Try again');
     end
end
%Creating a new matrix Aout to save the analysis output 
Aout=zeros(Yn,Xn);
%Comparing every cell of all files saved in A_Array to see if the sensor has
%passed or failed. 
for x = 1:Xn
    for y = 1:Yn
        for i = 1:N
            if A_Array(i,y,x)==1        %If sensor passed 
                Aout(y,x)=Aout(y,x)+1;  %Add one into the value of cell
            end
        end
    end
end
%Creating a new matrix Bout to represent the percentage of passing sensor in each cell 
Bout=zeros(Yn,Xn);
for y = 1:Yn
    for x = 1:Xn
        Bout(y,x)=Aout(y,x)/N*100;      %calculating percentage       
    end
end
Aout
Bout
%creating heatmap for the analysis
g = heatmap(Aout);
load('MyColormap','RtoG')   %Loading the created colorbar by Yekta
set(g,'Colormap',RtoG)      %Using the created colorbar by Yekta

g.ColorScaling = 'scaled';
g.ColorbarVisible = 'off';
g.GridVisible = 'off';
g.ColorMethod = 'count';
g.Title = 'Binary Analysis';
g.XLabel = 'X (inch)';
g.YLabel = 'Y (inch)';
g.FontColor = 'black';
g.CellLabelColor = 'black';
g.YDisplayLabels=string(yy);
g.XDisplayLabels=string(xx);
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
figure()
%creating heatmap for the percentage
h = heatmap(Bout);
load('MyColormap','RtoG')
set(h,'Colormap',RtoG)

h.ColorScaling = 'scaled';
h.ColorbarVisible = 'off';
h.GridVisible = 'off';
h.ColorMethod = 'count';
h.Title = 'Binary Percentage %';
h.XLabel = 'X (inch)';
h.YLabel = 'Y (inch)';
h.FontColor = 'black';
h.CellLabelColor = 'black';
h.YDisplayLabels=string(yy);
h.XDisplayLabels=string(xx);
%Adding the axis to the output to export into a excel file
Bout=[yy Bout]; %Add the y-axis to the first column
Bout=[xi;Bout]; %Add the x-axis to the last row
Aout=[yy Aout]; %Add the y-axis to the first column
Aout=[xi;Aout]; %Add the x-axis to the last row
xlswrite('Binary_Analysis.xlsx',Aout);
xlswrite('Binary_Percentage.xlsx',Bout);
