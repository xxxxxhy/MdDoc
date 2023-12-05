Dubins

~~~~matlab
function [ck,L,ssignr,xx,yy,tth]=dubins4(z0,zf,rhomin,obs0,flagkx2d,nz2d,zmin2d,zmax2d,mf)
% dubins曲线计算前处理及后处理
    lamht=100;
    n0=size(z0,1);nf=size(zf,1);
    % if n0~=nf;
    %     if n0<nf
    %         z0=repmat(z0_(1,:),nf,1);zf=zf_;
    %     elseif n0>nf
    %         zf=repmat(zf_(1,:),n0,1);z0=z0_;
    %     end
    % else
    %     z0=z0_;zf=zf_;
    % end
    n=max(n0,nf);
    signv=z0(:,5);
    xx=zeros(n,mf*3+1);
    yy=xx;
    tth=xx;
    ssignr=zeros(n,3);
    L=zeros(n,4);
    ck=zeros(n,1);
    iqj=signv>0;%前进
    [ck,L,ssignr,xx,yy,tth]=dubins(cto(z0(:,1:3),1),cto(zf(:,1:3),1),iqj,ck,L,ssignr,xx,yy,tth,rhomin,obs0,flagkx2d,nz2d,zmin2d,zmax2d,mf);
    %ck(iqj,:)=ck(iqj,:)+abs(zf(iqj,5)-z0(iqj,5))*lamht;%前进后退代价
    iht=signv<0;%后退
    [ck,L,ssignr,xx,yy,tth]=dubins(cto(zf(:,1:3),-1),cto(z0(:,1:3),-1),iht,ck,L,ssignr,xx,yy,tth,rhomin,obs0,flagkx2d,nz2d,zmin2d,zmax2d,mf);
    xx(iht,:)=flipud(xx(iht,:)')';
    yy(iht,:)=flipud(yy(iht,:)')';
    tth(iht,:)=flipud(tth(iht,:)')';
    L(iht,2:end)=flipud(L(iht,2:end)')';%L2是绝对值
    ssignr(iht,:)=flipud(ssignr(iht,:)')';
    ck=ck+abs(zf(:,5)-z0(:,5))*lamht;%前进后退代价
end

function [ck,L,ssignr,xx,yy,tth]=dubins(p1,p2,id,ck,L,ssignr,xx,yy,tth,rhomin,obs0,flagkx2d,nz2d,zmin2d,zmax2d,mf)
% dubins曲线计算前处理
    if min(numel(p1(id,:)),numel(p2(id,:)))>0
        [flagkx,L(id,:),ssignr(id,:),xx(id,:),yy(id,:),tth(id,:)]=dubins1(p1(id,1:3),p2(id,1:3),rhomin,obs0,flagkx2d,nz2d,zmin2d,zmax2d,mf);
        gmax=1e4;
        ck(id,:)=L(id,1)*rhomin+gmax*sum(~flagkx,2);
    else
        %ck=[];xx=[];yy=[];tth=[];ssignr=[];L=[];
    end
end

function [flagkx,L,ssignr,xxx,yyy,ttth,jopt]=dubins1(p1,p2,rhomin,obs0,flagkx2d,nz2d,zmin2d,zmax2d,mf)
    % rhomin=6;
    dx = p2(:,1) - p1(:,1);
    dy = p2(:,2) - p1(:,2);
    d = sqrt( dx.^2 + dy.^2 ) / rhomin;

    theta = mod(atan2( dy, dx ), 2*pi);
    alpha = mod((p1(:,3) - theta), 2*pi);
    beta  = mod((p2(:,3) - theta), 2*pi);
    L1 = LSL(alpha,beta,d);%L=[行驶距离  第一段长度 第二段长度 第三段长度]
    L2 = LSR(alpha,beta,d);
    L3 = RSL(alpha,beta,d);
    L4 = RSR(alpha,beta,d);
    L5 = RLR(alpha,beta,d);
    L6= LRL(alpha,beta,d);
    cost=[L1(:,1),L2(:,1),L3(:,1),L4(:,1),L5(:,1),L6(:,1)];
    n=size(d,1);
    [~,jopt]=min(cost,[],2);
    L=zeros(n,4);
    ssignr=zeros(n,3);
    signr=[1,0,1;
        1,0,-1;
        -1,0,1;
        -1,0,-1;
        -1,1,-1;
        1,-1,1;];
    for i=1:n
        j=jopt(i);
        switch j
            case 6
                L(i,:)=L6(i,:);
            case 1
                L(i,:)=L1(i,:);
            case 2
                L(i,:)=L2(i,:);
            case 3
                L(i,:)=L3(i,:);
            case 4
                L(i,:)=L4(i,:);
            case 5
                L(i,:)=L5(i,:);
        end
        ssignr(i,:)=signr(j,:);
    end
    m=mf;
    xxx=zeros(n,m*3+1);
    yyy=zeros(n,m*3+1);
    ttth=zeros(n,m*3+1);
    for i=1:n
        if L(i,1)>1e4
            continue;
        end
        si=L(i,2:4);
        th0=p1(i,3);
        x0=p1(i,1);
        y0=p1(i,2);
        signri=ssignr(i,:);
        dth1=si(1)*signri(1);
        th1=dth1+th0;
        dth2=si(2)*signri(2);
        th2=dth2+th1;
        dth3=si(3)*signri(3);
        th3=dth3+th2;
        x1=x0+signri(1)*rhomin*(sin(th1)-sin(th0));
        y1=y0-signri(1)*rhomin*(cos(th1)-cos(th0));
        tth1=th0+(th1-th0)/m*(1:m)';
        xx1=x0+signri(1)*rhomin*(sin(tth1)-sin(th0));
        yy1=y0-signri(1)*rhomin*(cos(tth1)-cos(th0));

        x2=x1+signri(2)*rhomin*(sin(th2)-sin(th1))+(signri(2)==0)*si(2)*rhomin*(cos(th2));
        y2=y1-signri(2)*rhomin*(cos(th2)-cos(th1))+(signri(2)==0)*si(2)*rhomin*(sin(th2));
        tth2=th1+(th2-th1)/m*(1:m)';
        ssi2=si(2)/m*(1:m)';
        xx2=x1+signri(2)*rhomin*(sin(tth2)-sin(th1))+(signri(2)==0)*ssi2*rhomin*(cos(th2));
        yy2=y1-signri(2)*rhomin*(cos(tth2)-cos(th1))+(signri(2)==0)*ssi2*rhomin*(sin(th2));

        %x3=x2+signri(3)*rhomin*(sin(th3)-sin(th2));
        %y3=y2-signri(3)*rhomin*(cos(th3)-cos(th2));
        tth3=th2+(th3-th2)/m*(1:m)';
        xx3=x2+signri(3)*rhomin*(sin(tth3)-sin(th2));
        yy3=y2-signri(3)*rhomin*(cos(tth3)-cos(th2));
        xx=[x0;xx1;xx2;xx3];%[x0,x1,x2,x3];
        yy=[y0;yy1;yy2;yy3];%[y0,y1,y2,y3];
        tth=[th0;tth1;tth2;tth3];%[th0,th1,th2,th3];
        xxx(i,:)=xx;
        yyy(i,:)=yy;
        ttth(i,:)=tth;
    end
    flagf=1;
    [flagkx]=checkcar3(xxx,yyy,ttth,flagkx2d,nz2d,zmin2d,zmax2d,flagf);
    %
    %     if 1
    %         [flagkx]=checkcar3([xx,yy,tth],flagkx2d,nz2d,zmin2d,zmax2d,flagf);
    %         cost(i)=L(i,1)+gmax*(sum(~flagkx));
    %     else
    %         dis=checkobs2(obs0,[ftoc([xx,yy,tth]),0*tth],1);
    %         cost(i)=L(i,1)+gmax*(sum(dis));
    %     end
    %     hold on;plot(xx,yy,'b-');
end

function L = LSL(alpha,beta,d)
    tmp0 = d + sin(alpha) - sin(beta);
    p_squared = 2 + (d.*d) -(2*cos(alpha - beta)) + (2*d.*(sin(alpha) - sin(beta)));
    L=zeros(size(d,1),4);
    i1=p_squared<0;L(i1,:)=inf;
    i2=p_squared>=0;p_squared=max(0,p_squared);
    tmp1 = atan2( (cos(beta)-cos(alpha)), tmp0 );
    t = mod((-alpha + tmp1 ), 2*pi);
    p = sqrt( p_squared );
    q = mod((beta - tmp1 ), 2*pi);
    li=[t+p+q,t,p,q];L(i2,:)=li(i2,:);
end


function L = LSR(alpha,beta,d)
    p_squared = -2 + (d.*d) + (2*cos(alpha - beta)) + (2*d.*(sin(alpha)+sin(beta)));
    L=zeros(size(d,1),4);
    i1=p_squared<0;L(i1,:)=inf;
    i2=p_squared>=0;p_squared=max(0,p_squared);
    p= sqrt( p_squared );
    tmp2 = atan2( (-cos(alpha)-cos(beta)), (d+sin(alpha)+sin(beta)) ) - atan2(-2.0, p);
    t= mod((-alpha + tmp2), 2*pi);
    q= mod((-mod((beta), 2*pi) + tmp2 ), 2*pi);
    li=[t+p+q,t,p,q];L(i2,:)=li(i2,:);
end

function L = RSL(alpha,beta,d)
    p_squared = (d.*d) -2 + (2*cos(alpha - beta)) - (2*d.*(sin(alpha)+sin(beta)));
    L=zeros(size(d,1),4);
    i1=p_squared<0;L(i1,:)=inf;
    i2=p_squared>=0;p_squared=max(0,p_squared);
    p    = sqrt( p_squared );
    tmp2 = atan2( (cos(alpha)+cos(beta)), (d-sin(alpha)-sin(beta)) ) - atan2(2.0, p);
    t    = mod((alpha - tmp2), 2*pi);
    q    = mod((beta - tmp2), 2*pi);
    li=[t+p+q,t,p,q];L(i2,:)=li(i2,:);
end

function L = RSR(alpha,beta,d)
    tmp0 = d-sin(alpha)+sin(beta);
    p_squared = 2 + (d.*d) -(2*cos(alpha - beta)) + (2*d.*(sin(beta)-sin(alpha)));
    L=zeros(size(d,1),4);
    i1=p_squared<0;L(i1,:)=inf;
    i2=p_squared>=0;p_squared=max(0,p_squared);
    tmp1 = atan2( (cos(alpha)-cos(beta)), tmp0 );
    t = mod(( alpha - tmp1 ), 2*pi);
    p = sqrt( p_squared );
    q = mod(( -beta + tmp1 ), 2*pi);
    li=[t+p+q,t,p,q];L(i2,:)=li(i2,:);
end

function L = RLR(alpha,beta,d)
    tmp_rlr = (6. - d.*d + 2*cos(alpha - beta) + 2*d.*(sin(alpha)-sin(beta))) / 8.;
    L=zeros(size(d,1),4);
    i1=abs(tmp_rlr) > 1;L(i1,:)=inf;
    i2=abs(tmp_rlr) <=1;tmp_rlr=max(-1,min(1,tmp_rlr));
    p = mod(( 2*pi - acos( tmp_rlr ) ), 2*pi);
    t = mod((alpha - atan2( cos(alpha)-cos(beta), d-sin(alpha)+sin(beta) ) + mod(p/2, 2*pi)), 2*pi);
    q = mod((alpha - beta - t + mod(p, 2*pi)), 2*pi);
    li=[t+p+q,t,p,q];L(i2,:)=li(i2,:);
end

function L = LRL(alpha,beta,d)
    tmp_lrl = (6. - d.*d + 2*cos(alpha - beta) + 2*d.*(- sin(alpha) + sin(beta))) / 8.;
    L=zeros(size(d,1),4);
    i1=abs(tmp_lrl) > 1;L(i1,:)=inf;
    i2=abs(tmp_lrl) <=1;tmp_lrl=max(-1,min(1,tmp_lrl));
    p = mod(( 2*pi - acos( tmp_lrl ) ), 2*pi);
    t = mod((-alpha - atan2( cos(alpha)-cos(beta), d+sin(alpha)-sin(beta) ) + p/2), 2*pi);
    q = mod((mod(beta, 2*pi) - alpha -t + mod(p, 2*pi)), 2*pi);
    li=[t+p+q,t,p,q];L(i2,:)=li(i2,:);
end