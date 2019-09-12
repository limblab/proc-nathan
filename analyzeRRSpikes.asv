%% set file name and load file into cds

    input_data.folderpath = 'C:\Users\nds4346\Desktop\Duncan_ringReporting\Duncan_RR_nathan_novision_20190905\';
    input_data.mapFileName = 'mapFileR:\limblab\lab_folder\Animal-Miscellany\Duncan_17L1\mapfiles\left S1 20190205\SN 6251-002087.cmp';
%     input_data.mapFileName = 'mapFileR:\limblab\lab_folder\Animal-Miscellany\Han_13B1\map files\Left S1\SN 6251-001459.cmp';

    input_data.date = '20190905';
    input_data.array = 'arrayLeftS1';
    input_data.monkey = 'monkeyDuncan';
    input_data.ranBy = 'ranByNathan';
    input_data.lab = 6;
    input_data.task = 'taskRR';

    pwd=cd;
    cd(input_data.folderpath)
    fileList = dir('*nev*');
    input_data.fileNum = 1;
  
    input_data.center_y = -33;
    input_data.center_x = 3;
    input_data.num_bins = 8;
%% convert file to cds and td
    td_all = [];
    for f = 1:numel(fileList)
        cds = commonDataStructure();
        cds.file2cds(strcat(input_data.folderpath,fileList(f).name),input_data.array,input_data.monkey,input_data.ranBy,...
            input_data.lab,input_data.mapFileName,input_data.task,'recoverPreSync','ignoreJumps','ignoreFilecat');
        cd(pwd);
    
        if(strcmpi(input_data.task,'RR'))
            params.event_list = {'otHoldTime';'goCueTime';'tgtDir';'bumpDir';'bumpTime';'bumpMagnitude';'stimTime';'stimCode';'catchTrial';'showOuterTarget'};
        else
            params.event_list = {'goCueTime';'bumpDir';'bumpTime';'bumpMagnitude';'stimTime';'stimCode'};
        end
        params.trial_results = {'R','F','A','I'};
        params.extra_time = [1,2];
        params.include_ts = 0;
        params.exclude_units = [0,255];
        td_temp = parseFileByTrial(cds,params);
%         td_temp = td_temp(~isnan([td_temp.catchTrial]));
        
        td_all = [td_all,td_temp];
    end
    
   input_data.tgt_width = mode([cds.trials.tgtWidth]);
   
   %% get bump trials, plot tgt_dir vs reach_dir. 
% plot percentage correct in bins around a polar plot

    td_bump_novision = td_all(~isnan([td_all.bumpDir]));
    
    % get pos at idx_endTime
    reach_angles_bump = zeros(numel(td_bump_novision),1);
    tgt_angles_bump = zeros(numel(td_bump_novision),1);
    is_rewarded = zeros(numel(td_bump_novision),1);
    for t = 1:numel(td_bump_novision)
        tgt_angles_bump(t) = td_bump_novision(t).target_direction;
        
        reach_angles_bump(t) = getReachAngle(td_bump_novision,t,[input_data.center_x,input_data.center_y]);
        is_rewarded(t) = td_bump_novision(t).result == 'R';
        is_rewarded(t) = abs(reach_angles(t) - tgt_angles(t))*180/pi < input_data.tgt_width/2;
    end
%     f=figure(); hold on
%     f.Name = [input_data.monkey(7:end),'_',input_data.date,'_bump_reachVsTgt'];
%     
%     % unity black line
%     plot([-180,180],[-180,180],'k--','linewidth',2);
%     % error tolerance lines (4 to compensate for wrap arounds)
%     plot([-180,180],[-180,180]+input_data.tgt_width/2,'r--','linewidth',2);
%     plot([-180,180],[-180,180]+360-input_data.tgt_width/2,'r--','linewidth',2);
%     
%     plot([-180,180],[-180,180]-input_data.tgt_width/2*180/180,'r--','linewidth',2);
%     plot([-180,180],[-180,180]-360+input_data.tgt_width/2,'r--','linewidth',2);
%     
%     % plot reach vs tgt
%     plot(tgt_angles_bump*180/pi,reach_angles_bump*180/pi,'.','markersize',16)
%     % format
%     xlim([-180,180]); ylim([-180,180]);
%     formatForLee(gcf); set(gca,'fontsize',14)
%     xlabel('Tgt dir'); ylabel('Reach dir');
%     
%     % get percent correct in bins around a circle
%     bin_edges = [-pi:2*pi/input_data.num_bins:pi];
%     percent_correct = zeros(numel(bin_edges)-1,1);
%     for i = 1:numel(bin_edges)-1
%         trial_mask = tgt_angles_bump >= bin_edges(i) & tgt_angles_bump < bin_edges(i+1);
%         percent_correct(i) = sum(is_rewarded(trial_mask))/sum(trial_mask);
%     end
%     
%     f=figure();
%     f.Name = [input_data.monkey(7:end),'_',input_data.date,'_bump_percentCorrect'];
%     polarplot([bin_edges(1:end-1),bin_edges(1)]+mode(diff(bin_edges))/2,[percent_correct;percent_correct(1)],...
%         '--.','markersize',20,'linewidth',2)
%     f.Children(1).RLim = [0,1]; % radius limits from 0 to 1
%     ax = gca;
%     ax.ThetaTickLabel = {'0','30','60','90','120','150','180','-150','-120','-90','-60','-30'};
%     %% Change bin size


%% Bin size
td_bump_vision = binTD(td_bump_vision, 20);
 %% Git rid of non-reward trials
reward_novision = td_bump_novision;

for n = 1:numel(td_bump_novision)
    if reward_novision(n).result == 'R'
        reward_novision(n).result = 1;        
    else
        reward_novision(n).result = 0;
    end
end

A = [reward_novision.result];
B = find(A==0);
reward_novision(B) = [];

%% create a speed component
for n=1:numel(reward_novision)
    reward_novision(n).speed = sqrt((reward_novision(n).vel(:,1)).^2 + (reward_novision(n).vel(:,2)).^2);
end

%% align with movement
moveParams.which_field = 'speed';
moveParams.start_idx = 'idx_bumpTime';
moveParams.end_idx = 'idx_endTime'; 
moveParams.which_method = 'peak';
moveParams.s_thresh = 10;
moveParams.min_ds = 1.9;

reward_vision= getMoveOnsetAndPeak(reward_vision,moveParams);

%% Trim again to have movement onset to be the same
for n = 1:numel(reward_vision)
reward_vision(n) = trimTD(reward_vision(n), {reward_vision(n).idx_bumpTime+ 10},{(reward_vision(n).idx_bumpTime + 20)});
end

%% Smooth spikes
smoothParams.signals ={'pos','vel','force'};
smoothParams.calc_rate= false;

reward_vision= smoothSignals(reward_vision,smoothParams);

smoothParams.signals ={'LeftS1_spikes'};
smoothParams.calc_rate = true;

reward_vision = smoothSignals(reward_vision,smoothParams);

%% Plot mean of all neurons in each direction
unique_bumpDir= unique([reward_vision.bumpDir]);

window = [0,10];
  numtrials = zeros(8,1,diff(window)+1);
  numspikes= zeros(8,1,diff(window)+1);


 f=figure(); hold on
    f.Name = [input_data.monkey(7:end),'_',input_data.date,'_meanFiringRateEachDir'];
for n= 1:numel(reward_vision)
    if numel(reward_vision(n).speed)>window(2)
        A=reward_vision(n).bumpDir==unique_bumpDir;
        i=find(A==1);
        numtrials(i,1,:) = numtrials(i,1,:) + 1;
        idx_window = reward_vision(n).idx_movement_on + window;
        numspikes(i,1,:) = numspikes(i,1,:) + reshape(reward_vision(n).LeftS1_spikes(idx_window(1):idx_window(2),1),1,1,diff(window)+1);
    end
end

for n= 1:numel(reward_vision)
    if numel(reward_vision(n).speed)>window(2)
        A=reward_vision(n).bumpDir==unique_bumpDir;
        i=find(A==1);
        Z= squeeze(numspikes(i,1,:)./numtrials(i,1,:));
        subplot(8,1,i)
        plot(Z,'LineWidth',2,'Color','r')
        ylim([0 30])
        hold on
    end
end
    
    window = [0,10];
  numtrials = zeros(8,1,diff(window)+1);
  numspikes= zeros(8,1,diff(window)+1);

    
for n= 1:numel(reward_novision)
    if numel(reward_novision(n).speed)>window(2)
        A=reward_novision(n).bumpDir==unique_bumpDir;
        i=find(A==1);
        numtrials(i,1,:) = numtrials(i,1,:) + 1;
        idx_window = reward_novision(n).idx_movement_on + window;
        numspikes(i,1,:) = numspikes(i,1,:) + reshape(reward_novision(n).LeftS1_spikes(idx_window(1):idx_window(2),1),1,1,diff(window)+1);
    end
end

for n= 1:numel(reward_novision)
    if numel(reward_novision(n).speed)>window(2)
        A=reward_novision(n).bumpDir==unique_bumpDir;
        i=find(A==1);
        Z= squeeze(numspikes(i,1,:)./numtrials(i,1,:));
        subplot(8,1,i)
        plot(Z,'LineWidth',2,'Color','b')
        ylim([0 30])
        hold on
    end
end
%% Plot individual neurons at once across 8 directions
%x is the individual neuron
 f=figure(); hold on
    f.Name = [input_data.monkey(7:end),'_',input_data.date,'_singleNeuronEachDir'];
 
for n = 1:numel(reward_vision)
      if numel(reward_vision(n).speed)>50
        A=reward_vision(n).bumpDir==unique_bumpDir;
        i=find(A==1);
        subplot(8,1,i)
        plot(reward_vision(n).LeftS1_spikes(:,x), 'Color', 'b')
        ylim([0 100])
        hold on
      end
end

for n = 1:numel(reward_novision)
      if numel(reward_novision(n).speed)>50
        A=reward_novision(n).bumpDir==unique_bumpDir;
        i=find(A==1);
        subplot(8,1,i)
        plot(reward_novision(n).LeftS1_spikes(:,x), 'Color', 'r')
        ylim([0 100])
        hold on
      end
end


%% Plot each mean firing rate of each neuron

for n = 1:numel(reward_vision)
    if numel(reward_vision(n).speed)>50
        A=reward_vision(n).bumpDir==unique_bumpDir;
        i=find(A==1);
    for x = 1:numel(reward_vision(n).LeftS1_spikes(1,:))
        y = sum(reward_vision(n).LeftS1_spikes(:,x))/numel(reward_vision(n).speed);
        subplot(8,1,i)
        plot(x,y, 'ob')
        hold on
    end
    end
end
       
 %% compare firing rate of vision vs no vision for each direction
 
 %no vision
 n = 1;
spikes_novision = zeros(numel(reward_novision(n).speed),numel(reward_novision(n).LeftS1_spikes(1,:)),numel(unique_bumpDir));
spike_sum_novision = zeros(1, numel(reward_novision(n).LeftS1_spikes(1,:)), numel(unique_bumpDir));
spike_trialNoVision = zeros(1, numel(reward_novision(n).LeftS1_spikes(1,:)), numel(unique_bumpDir));

for n = 1:numel(reward_novision)
    if numel(reward_novision(n).speed) > window(2)
        for bump = 1:numel(unique_bumpDir)
            if reward_novision(n).bumpDir == unique_bumpDir(bump)
               spikes_novision(:,:,bump) = spikes_novision(:,:,bump) + reward_novision(n).LeftS1_spikes;
               spike_trialNoVision(1,:,bump) = spike_trialNoVision(1,:,bump) + 1;
            end
        end
    end
end

for bump = 1:numel(unique_bumpDir)
      spike_sum_novision(1,:,bump) = sum(spikes_novision(:,:,bump));
end

    novision_plot = ((spike_sum_novision./spike_trialNoVision)/numel(reward_novision(n).speed))/(reward_novision(n).bin_size);


% vision
n = 1;
spikes_vision = zeros(numel(reward_vision(n).speed),numel(reward_vision(n).LeftS1_spikes(1,:)),numel(unique_bumpDir));
spike_sum_vision = zeros(1, numel(reward_vision(n).LeftS1_spikes(1,:)), numel(unique_bumpDir));
spike_trialVision = zeros(1, numel(reward_vision(n).LeftS1_spikes(1,:)), numel(unique_bumpDir));

for n = 1:numel(reward_vision)
    if numel(reward_vision(n).speed) > window(2)
        for bump = 1:numel(unique_bumpDir)
            if reward_vision(n).bumpDir == unique_bumpDir(bump)
                spikes_vision(:,:,bump) = spikes_vision(:,:,bump) + reward_vision(n).LeftS1_spikes;
                spike_trialVision(1,:,bump) = spike_trialVision(1,:,bump) + 1;
            end
        end
    end
end

for bump = 1:numel(unique_bumpDir)
      spike_sum_vision(1,:,bump) = sum(spikes_vision(:,:,bump));
end
vision_plot =((spike_sum_vision./spike_trialVision)/numel(reward_vision(n).speed))/(reward_vision(n).bin_size);

%% plot
    f=figure(); hold on
    f.Name = [input_data.monkey(7:end),'_',input_data.date,'_FiringRatesForVisionVsNoVision'];

for bump = 1:numel(unique_bumpDir)
    subplot(4,2,bump)
    y=x;
    x=[0:200];
    plot(y,'--k')
    hold on
    plot(novision_plot(:,:,bump),vision_plot(:,:,bump), '.b');
    xlim([0 200])
    ylim([0 200])
    xlabel('Without Vision')
    ylabel('With Vision')
    title('Ring Reporting Mean Firing Rates')
    
 
end


%% Plot mean firing rate of each individual neuron across each direction with vision/non vision on same plot
% 8 figures w 71 plots
n = 1;

for bump = 1: numel(unique_bumpDir)
    f=figure(); hold on
    f.Name = [input_data.monkey(7:end),'_',input_data.date,'_AllNeurons'];
    for x = 1:numel(reward_vision(1).LeftS1_spikes(1,:))
        subplot(8,9,x)
        plot((spikes_vision(:,x,bump)/spike_trialVision(:,x,bump)/(reward_vision(n).bin_size)),'r')
        hold on 
        plot((spikes_novision(:,x,bump)/spike_trialNoVision(:,x,bump)/(reward_novision(n).bin_size)),'b')
        xlim([0 11])
        ylim([0 100])
    end
end





