---
layout: post
title: "Coate and Loury Simulation"
date: 2018-08-06
---

Ever wondered how you would generate the images in the seminal paper on statistical discrimination, [Coate and Loury (1993)](https://www.jstor.org/stable/2117558)?

The following code is a simulation of the model and generates the paper's images -- existence of multiple equilibria and benign+patronizing equilibria in the presence of affirmative action. If I find time in the near future, I will comment
this code to explain the model alongside the code.

Here are the images that are generated:

## Existence of Multiple Equilibria

<div><img src="https://namsaxena.github.io/images/PatronizingEqAA_decrho.png" /></div> 

```markdown
%%
%Section 1: Implements Coate and Loury -- finds fixed point
%Section 2: Coate and Loury with AA -- non-decreasing rho_hat, patronizing
%Section 3: Coate and Loury with AA -- non-decreasing rho_hat, benign
%Section 4: Coate and Loury with AA -- decreasing rho_hat
 
clear all;
close all;
clc;
 
%%
%Section1: Implementing Coate and Loury - straightforward
%set constants
theta=0:0.01:1;
theta_q=0.4;
theta_u=0.95;
omega=5;
chi_q=6;
chi_u=5;
pi_b=0:0.01:1;
temp=zeros(101,1);
 
    for i=1:length(pi_b)
        f_q=pdf('Beta',theta,2,2);
        f_u=pdf('Beta',theta,2,5);
        %f_q=unifpdf(theta,theta_q,1);
        %f_u=unifpdf(theta,0,theta_u);
        lr=f_u./f_q;
        val=pi_b(i);
        pi=(1-pi_b(i))/(pi_b(i));
        r=chi_q/chi_u;
        %for every value of pi, check&store the thetas s.t. return>=lr*belief
        mult=pi.*lr;
        ret(1:101)=r;
        %standard:choose the minimum theta such that r>=mult
        mat = [ret;mult;theta]';
        subset=mat(mat(:,1)>=mat(:,2),3);
        standard=min(subset);
        incprob=[unifcdf(standard,0,theta_u)-unifcdf(standard,theta_q,1)];
        %now,calculate net benefit
        %benefit=omega*[unifcdf(standard,0,theta_u)-unifcdf(standard,theta_q,1)];
        benefit=omega*[cdf('Beta',standard,2,5)-cdf('Beta',standard,2,2)]
        %now that I have benefit, I can calculate proportion of population
        pi_ww=cdf('Lognormal',benefit,0.5,0.11); 
        %pi_ww=unifcdf(benefit,0,1);
        if ~isempty(pi_ww);
            outpi(i) = pi_ww;
            outs(i) = standard;
        else
            outpi(i) = NaN;
            outs(i) = NaN;
        end;
        if pi_ww==pi_b(i)
           temp(i)=pi_ww;
        end
    end    
 
 
%h1=plot(pi_b,outpi);hold on;
%h2=plot(pi_b,pi_b)
%title("Fixed Point")
figure;
h3=plot(outs,outpi);hold on;
h4=plot(outs,pi_b)
xlim([0 1])
title("Multiple Equilbria")
%plot(pi_b,outs);
``` 
 
## Patronizing Equilibria Under Affirmative Action
<div><img src="https://namsaxena.github.io/images/PatronizingEqAA_nondecrho.png" /></div> 

```markdown
%%
%Section2: Coate and Loury with AA, non-decreasing rho_hat
%set constants
sd=0.2;
theta=0:0.01:1;
omega=5;
chi_q=6;
chi_u=5;
pi_b=0:0.01:1;
gamma=0.2819;
lambda=0.94;
 
%Define useful functions
f_q = @(s) pdf('Beta',s,2,2);
f_u = @(s) pdf('Beta',s,2,5);
F_q = @(s) cdf('Beta',s,2,2);
F_u = @(s) cdf('Beta',s,2,5);
 
    for i=1:length(pi_b)
        f_qconst=f_q(theta);
        f_uconst=f_u(theta);
        %f_q=unifpdf(theta,theta_q,1);
        %f_u=unifpdf(theta,0,theta_u);
        lr=f_uconst./f_qconst;
        val=pi_b(i);
        pi=(1-pi_b(i))/(pi_b(i));
         
        %white people
        r_w=(chi_q-(gamma/lambda))/(chi_u+(gamma/lambda));
        %for every value of pi, check&store the thetas s.t. return>=lr*belief
        mult_w=pi.*lr;
        ret(1:101)=r_w;
        %standard:choose the minimum theta such that r>=mult
        mat=[ret;mult_w;theta]';
        subset=mat(mat(:,1)>=mat(:,2),3);
        standard=min(subset);
        %now,calculate net benefit
        %benefit=omega*[unifcdf(standard,0,theta_u)-unifcdf(standard,theta_q,1)];
        benefit=omega*(F_q(standard)-F_u(standard));
        %now that I have benefit, I can calculate proportion of population
        pi_ww=cdf('Lognormal',benefit,0.5,sd); 
        %pi_ww=unifcdf(benefit,0,1);
        if ~isempty(pi_ww)
            outpi_w(i) = pi_ww;
            outs_w(i) = standard;
        else
            outpi_w(i) = NaN;
            outs_w(i) = NaN;
        end
         
        %black people
        r_b=(chi_q+(gamma/(1-lambda)))/(chi_u-(gamma/(1-lambda)));
         %for every value of pi, check&store the thetas s.t. return>=lr*belief
        mult_b=pi.*lr;
        ret(1:101)=r_b;
        %standard:choose the minimum theta such that r>=mult
        mat=[ret;mult_b;theta]';
        subset=mat(mat(:,1)>=mat(:,2),3);
        standard=min(subset);
        %now,calculate net benefit
        %benefit=omega*[unifcdf(standard,0,theta_u)-unifcdf(standard,theta_q,1)];
        benefit=omega*(F_q(standard)-F_u(standard));
        %now that I have benefit, I can calculate proportion of population
        pi_bb=cdf('Lognormal',benefit,0.5,sd); 
        %pi_ww=unifcdf(benefit,0,1);
        if ~isempty(pi_ww)
            outpi_b(i) = pi_bb;
            outs_b(i) = standard;
        else
            outpi_b(i) = NaN;
            outs_b(i) = NaN;
        end
         
        %gamma=0
        r=chi_q/chi_u;
        %for every value of pi, check&store the thetas s.t. return>=lr*belief
        mult=pi.*lr;
        ret(1:101)=r;
        %standard:choose the minimum theta such that r>=mult
        mat = [ret;mult;theta]';
        subset=mat(mat(:,1)>=mat(:,2),3);
        standard=min(subset);
        %now,calculate net benefit
        %benefit=omega*[unifcdf(standard,0,theta_u)-unifcdf(standard,theta_q,1)];
        benefit=omega*(F_u(standard)-F_q(standard));
        %now that I have benefit, I can calculate proportion of population
        pi_ww=cdf('Lognormal',benefit,0.5,sd); 
        %pi_ww=unifcdf(benefit,0,1);
        if ~isempty(pi_ww);
            outpi(i) = pi_ww;
            outs(i) = standard;
        else
            outpi(i) = NaN;
            outs(i) = NaN;
        end;
        if pi_ww==pi_b(i)
           temp(i)=pi_ww;
        end
         
    end    
 
%find intersections
[x0,y0]=intersections(outs,outpi,outs_b,pi_b);
[x1,y1]=intersections(outs,outpi,outs_w,pi_b);
sb=x0(1)
pib=y0(1)
sw=x1(1)
piw=y1(1)
benefit_b=omega*(cdf('Beta',sb,2,5)-cdf('Beta',sb,2,2));
benefit_w=omega*(cdf('Beta',sw,2,5)-cdf('Beta',sw,2,2));
rho_hat_b=(cdf('Lognormal',benefit_b,0.5,sd))*(1-cdf('Beta',sb,2,2))+(1-(cdf('Lognormal',benefit_b,0.5,sd)))*(1-cdf('Beta',sb,2,5))
rho_hat_w=(cdf('Lognormal',benefit_w,0.5,sd))*(1-cdf('Beta',sw,2,2))+(1-(cdf('Lognormal',benefit_w,0.5,sd)))*(1-cdf('Beta',sw,2,5))
 
%Graph
%h1=plot(pi_b,outpi);hold on;
%h2=plot(pi_b,pi_b)
%title("Fixed Point"
figure
set(groot,'DefaultAxesColorOrder',[0 0 0])
whitebg([1 1 1])
subplot(2,1,2)
h3=plot(outs,outpi,'k');hold on;
h4=plot(outs_b,pi_b,'k');hold on;
h5=plot(outs_w,pi_b,'k');hold on;
line([0.2883;0.2883],[0,1],'linestyle','--');hold on;
line([0.2014;0.2014],[0,1],'linestyle','--');
set([h3 h4 h5],'Linewidth',1.5)
legend('WW','EE_b','EE_w','Location','northeast')
xlim([0 1]);
xlim([0 1])
title("Patronizing Equilbrium under Affirmative Action")
 
%plot rho_hat
F_q=cdf('Beta',outs_w,2,2);
F_u=cdf('Beta',outs_w,2,5);
 
diff=F_q-F_u;
 
benefit=omega.*[F_u-F_q];
prop=(0.95*cdf('Lognormal',benefit,0.5,sd)+0.05);
rho_hat=prop.*[1-F_q]+(1-prop).*[1-F_u];
subplot(2,1,1)
h6=plot(outs_w,rho_hat,'k');hold on;
set([h6],'Linewidth',1.5)
line([0.2883;0.2883],[0,1],'linestyle','--');hold on;
line([0.2014;0.2014],[0,1],'linestyle','--');
line([0;0.2883],[0.68;0.68],'linestyle','--')
xlim([0 1]);
title('$$\hat{\rho}$$','Interpreter','Latex')
str='$$\pi_b=0.068, s_b=0.20, \pi_w=0.632, s_w=0.288,\hat{\rho}=0.678$$';
text(0.25,-1.6,str,'Interpreter','latex')
saveas(gcf,'PatronizingEqAA_nondecrho.png')
 
```
 

## Benign Equilibria Under Affirmative Action

<div><img src="https://namsaxena.github.io/images/BenignEqAA_nondecrho.png" alt="benign" /></div> 

```markdown
%%
%Section3: Coate and Loury with AA, non-decreasing rho_hat->Benign eq
%set constants
sd=0.2;
theta=0:0.01:1;
omega=5;
chi_q=6;
chi_u=5;
pi_b=0:0.01:1;
gamma=0;
lambda=0.94;
 
%Define useful functions
f_q = @(s) pdf('Beta',s,2,2);
f_u = @(s) pdf('Beta',s,2,5);
F_q = @(s) cdf('Beta',s,2,2);
F_u = @(s) cdf('Beta',s,2,5);
 
    for i=1:length(pi_b)
        f_qconst=f_q(theta);
        f_uconst=f_u(theta);
        %f_q=unifpdf(theta,theta_q,1);
        %f_u=unifpdf(theta,0,theta_u);
        lr=f_uconst./f_qconst;
        val=pi_b(i);
        pi=(1-pi_b(i))/(pi_b(i));
         
        %white people
        r_w=(chi_q-(gamma/lambda))/(chi_u+(gamma/lambda));
        %for every value of pi, check&store the thetas s.t. return>=lr*belief
        mult_w=pi.*lr;
        ret(1:101)=r_w;
        %standard:choose the minimum theta such that r>=mult
        mat=[ret;mult_w;theta]';
        subset=mat(mat(:,1)>=mat(:,2),3);
        standard=min(subset);
        %now,calculate net benefit
        %benefit=omega*[unifcdf(standard,0,theta_u)-unifcdf(standard,theta_q,1)];
        benefit=omega*(F_q(standard)-F_u(standard));
        %now that I have benefit, I can calculate proportion of population
        pi_ww=cdf('Lognormal',benefit,0.5,sd); 
        %pi_ww=unifcdf(benefit,0,1);
        if ~isempty(pi_ww)
            outpi_w(i) = pi_ww;
            outs_w(i) = standard;
        else
            outpi_w(i) = NaN;
            outs_w(i) = NaN;
        end
         
        %black people
        r_b=(chi_q+(gamma/(1-lambda)))/(chi_u-(gamma/(1-lambda)));
         %for every value of pi, check&store the thetas s.t. return>=lr*belief
        mult_b=pi.*lr;
        ret(1:101)=r_b;
        %standard:choose the minimum theta such that r>=mult
        mat=[ret;mult_b;theta]';
        subset=mat(mat(:,1)>=mat(:,2),3);
        standard=min(subset);
        %now,calculate net benefit
        %benefit=omega*[unifcdf(standard,0,theta_u)-unifcdf(standard,theta_q,1)];
        benefit=omega*(F_q(standard)-F_u(standard));
        %now that I have benefit, I can calculate proportion of population
        pi_bb=cdf('Lognormal',benefit,0.5,sd); 
        %pi_ww=unifcdf(benefit,0,1);
        if ~isempty(pi_ww)
            outpi_b(i) = pi_bb;
            outs_b(i) = standard;
        else
            outpi_b(i) = NaN;
            outs_b(i) = NaN;
        end
         
        %gamma=0
        r=chi_q/chi_u;
        %for every value of pi, check&store the thetas s.t. return>=lr*belief
        mult=pi.*lr;
        ret(1:101)=r;
        %standard:choose the minimum theta such that r>=mult
        mat = [ret;mult;theta]';
        subset=mat(mat(:,1)>=mat(:,2),3);
        standard=min(subset);
        %now,calculate net benefit
        %benefit=omega*[unifcdf(standard,0,theta_u)-unifcdf(standard,theta_q,1)];
        benefit=omega*(F_u(standard)-F_q(standard));
        %now that I have benefit, I can calculate proportion of population
        pi_ww=cdf('Lognormal',benefit,0.5,sd); 
        %pi_ww=unifcdf(benefit,0,1);
        if ~isempty(pi_ww);
            outpi(i) = pi_ww;
            outs(i) = standard;
        else
            outpi(i) = NaN;
            outs(i) = NaN;
        end;
        if pi_ww==pi_b(i)
           temp(i)=pi_ww;
        end
         
    end    
 
%find intersections
[x0,y0]=intersections(outs,outpi,outs_b,pi_b);
[x1,y1]=intersections(outs,outpi,outs_w,pi_b);
sb=x0(2)
pib=y0(2)
sw=x1(1)
piw=y1(1)
benefit_b=omega*(cdf('Beta',sb,2,5)-cdf('Beta',sb,2,2));
benefit_w=omega*(cdf('Beta',sw,2,5)-cdf('Beta',sw,2,2));
rho_hat_b=(cdf('Lognormal',benefit_b,0.5,sd))*(1-cdf('Beta',sb,2,2))+(1-(cdf('Lognormal',benefit_b,0.5,sd)))*(1-cdf('Beta',sb,2,5))
rho_hat_w=(cdf('Lognormal',benefit_w,0.5,sd))*(1-cdf('Beta',sw,2,2))+(1-(cdf('Lognormal',benefit_w,0.5,sd)))*(1-cdf('Beta',sw,2,5))
 
%Graph
%h1=plot(pi_b,outpi);hold on;
%h2=plot(pi_b,pi_b)
%title("Fixed Point"
figure
set(groot,'DefaultAxesColorOrder',[0 0 0])
whitebg([1 1 1])
subplot(2,1,2)
h3=plot(outs,outpi,'k');hold on;
h4=plot(outs_b,pi_b,'k');hold on;
h5=plot(outs_w,pi_b,'k');hold on;
line([0.2837;0.2837],[0,1],'linestyle','--');hold on;
line([0.637;0.637],[0,1],'linestyle','--');
set([h3 h4 h5],'Linewidth',1.5)
legend('WW','EE_b','EE_w','Location','northeast')
xlim([0 1]);
xlim([0 1])
title("Benign Equilbrium under Affirmative Action")
 
%plot rho_hat
F_q=cdf('Beta',outs_w,2,2);
F_u=cdf('Beta',outs_w,2,5);
 
diff=F_q-F_u;
 
benefit=omega.*[F_u-F_q];
prop=(0.95*cdf('Lognormal',benefit,0.5,sd)+0.05);
rho_hat=prop.*[1-F_q]+(1-prop).*[1-F_u];
subplot(2,1,1)
h6=plot(outs_w,rho_hat,'k');hold on;
set([h6],'Linewidth',1.5)
line([0.2837;0.2837],[0,1],'linestyle','--');hold on;
line([0.637;0.637],[0,1],'linestyle','--');
line([0;0.2837],[0.667;0.667],'linestyle','--')
line([0;0.637],[0.0732;0.0732],'linestyle','--')
xlim([0 1]);
title('$$\hat{\rho}$$','Interpreter','Latex')
str='$$(\pi_1=0.606,s_1=0.284,\hat{\rho_2}=0.667), (\pi_2=0.173, s_2=0.637,\hat{\rho_1}=0.073)$$';
text(0.2,-1.6,str,'Interpreter','latex')
saveas(gcf,'BenignEqAA_nondecrho.png')
``` 

