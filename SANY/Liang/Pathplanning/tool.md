### 障碍物点云和墙壁直线二维栅格生成

``` matlab
function [pp,flagobs]=fobssg2d(obs,wall,rsafe,zmin,zmax,nz)
% fobssg2d 根据障碍点云与墙壁直线，结合安全距离和有效范围，生成障碍栅格
%   obs 障碍物点云
%   wall 墙壁直线
%   rsafe 安全距离
%   zmin zmax 区域范围
%   nz 栅格个数
    xmin=zmin(1);ymin=zmin(2);
    xmax=zmax(1);ymax=zmax(2);
    %生成栅格并计算是否被障碍所占据
    n1=nz(1);n2=nz(2);
    dx=(xmax-xmin)/n1;x=xmin+(0.5:n1)*dx;
    dy=(ymax-ymin)/n2;y=ymin+(0.5:n2)*dy;
    [ys,xs]=meshgrid(y,x);%j=3;li=ys(:,j,:);li=li(:);figure;plot(li-y(j))
    pp=zeros(n1*n2,3);
    pp(1:n1*n2,1)=xs(:);pp(1:n1*n2,2)=ys(:);
    [dobs,dwall]=fdisobs(pp,obs,wall);
    if numel(dwall)>0
        flagobs=(dobs>rsafe)&(dwall>rsafe*1);
    else
        flagobs=dobs>rsafe;
    end
end

function [d1,d2]=fdisobs(pp,obs,wall)
% 检查点障碍和障碍点、墙壁之间的最短距离
%   pp 二维栅格
%   obs 障碍物点云
%   wall 墙壁直线
    n=size(pp,1);
    x1=pp(:,1);y1=pp(:,2);
    dd1=1000*ones(n,max(1,size(obs,1)));
    for j=1:size(obs,1)
        erx=x1-obs(j,1);ery=y1-obs(j,2);
        dj=sqrt(erx.^2+ery.^2);
        dd1(:,j)=dj;
    end
    d1=min(dd1,[],2);
    dd2=1000*ones(n,size(wall,1));
    for j=1:size(wall,1)
    dd2(:,j)=fdispl(wall(j,1:2),wall(j,3:4),pp(:,1:2));
    end
    d2=min(dd2,[],2);
end

function [dopt]=fdispl(za,zb,obs)
%   计算点到线的最短距离
%   计算从za到zb的连线与障碍点obs之间的最短距离
%   za zb 障碍墙壁直线的两端坐标
%   obs 二维栅格点
    dx=zb(:,1)-za(:,1);
    dy=zb(:,2)-za(:,2);
    dxo=obs(:,1)-za(:,1);
    dyo=obs(:,2)-za(:,2);
    li=dx.^2+dy.^2;
    ts=(dxo.*dx+dyo.*dy)./li;
    era=[za(:,1)-obs(:,1),za(:,2)-obs(:,2)];da=sqrt(sum(era.^2,2));
    erb=[zb(:,1)-obs(:,1),zb(:,2)-obs(:,2)];db=sqrt(sum(erb.^2,2));
    eropt=era+[ts.*dx,ts.*dy];dopt=sqrt(sum(eropt.^2,2));
    id=(ts>0)&(ts<1);id1=~id;
    % n=max(size(zb,1),size(za,1));
    dopt(id1)=min(da(id1),db(id1));
end
```


### 障碍物点云和墙壁直线，三维障碍栅格生成

``` matlab
function [pp,flagkx3d,nz,dz]=fobssg3d1(zmin,zmax,nz,flagkxsg2d,zmin2d,zmax2d,nz2d)
% 生成x,y,th三个维度上的栅格点
%   zmin,zmax 定义有效范围
%   nz 定义栅格各个维度上的个数
%   flagkxsg2d 二维栅格的有效位
%   zmin2d,zmax2d 定义二维平面上的有效范围，nz2d二维栅格的有效个数

% [zmin2d,zmax2d]必须涵盖[zmin,zmax]的x,y维度
    xmin=zmin(1);ymin=zmin(2);thmin=zmin(3);
    xmax=zmax(1);ymax=zmax(2);thmax=zmax(3);
    %生成栅格并计算是否被障碍所占据
    n1=nz(1);n2=nz(2);n3=nz(3);
    dx=(xmax-xmin)/n1;x=xmin+(0.5:n1)*dx;
    dy=(ymax-ymin)/n2;y=ymin+(0.5:n2)*dy;
    dth=(thmax-thmin)/n3;th=thmin+(0.5:n3)*dth;

    [ys,xs,ths]=meshgrid(y,x,th);%j=3;li=ys(:,j,:);li=li(:);figure;plot(li-y(j))
    % N=n1*n2*n3+2;
    pp=zeros(n1*n2*n3,3);
    pp(1:n1*n2*n3,1)=xs(:);pp(1:n1*n2*n3,2)=ys(:);pp(1:n1*n2*n3,3)=ths(:);
    flagkx3d=ones(size(pp,1),1);
    [ki,ir]=getindex2d(pp(:,1),pp(:,2),nz2d,zmin2d,zmax2d);
    flagkx3d(ir<=0)=0;
    flagkx3d(ir>0)=flagkx3d(ir>0)&flagkxsg2d(ki(ir>0));
    dz=[dx,dy,dth];
end

function [k,ir]=getindex2d(xx,yy,nz,zmin,zmax)
% 根据二维计算栅格索引
    dz=(zmax(1:2)-zmin(1:2))./nz(1:2);
    id1=ceil((xx-zmin(1))/dz(1));
    id2=ceil((yy-zmin(2))/dz(2));
    ir=(id1>=1)&(id1<=nz(1))&(id2>=1)&(id2<=nz(2));
    k=id1+(id2-1)*nz(1);
end
```

### 结合车体进行点云过滤，确定可行范围

```matlab
function [flagkx1]=checkcar3_1(z,flagkxsg2d,nzsg2d,zminsg2d,zmaxsg2d,flagf)
% 结合三维栅格点，判断在各个位置以各个航向角情况下的车体与障碍的交叉情况，并生成可行范围
%   z 包含航向角的三维栅格
%   flagkxsg2d 二维栅格有效位
    x=z(:,1);y=z(:,2);thf=z(:,3);
    lf=4.5;
    lr=4+0;
    l=1.71;%前后车中心和铰接点的距离
    m=10;
    ds=(lf+lr)/m;
    ss=-lr-flagf*l+ds*(0:m)';
    flagkx=ones(size(x))>0;
    for i=1:m+1
        xi=x+ss(i)*cos(thf);
        yi=y+ss(i)*sin(thf);
        [ki,ir]=getindex2d(xi,yi,nzsg2d,zminsg2d,zmaxsg2d);
        if 0%用于可视化分析
            li=flagkxsg2d(ki);
            if li>0
                hold on;plot(xi,yi,'ro')
            else
                hold on;plot(xi,yi,'bo')
            end
        end
        flagkx(ir<=0)=0;
        flagkx(ir>0)=flagkx(ir>0)&flagkxsg2d(ki(ir>0));
    end
    flagkx1=reshape(flagkx,size(x,1),size(x,2));
end
```

### 杜宾斯曲线

```matlab
function [ck,L,ssignr,xx,yy,tth]=dubins4(z0,zf,rhomin,obs0,flagkx2d,nz2d,zmin2d,zmax2d,mf)
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
```
