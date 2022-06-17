# Buffer Overflow

![](https://scontent.xx.fbcdn.net/v/t1.15752-9/287382801_1012708796106310_3393263325642954415_n.png?stp=dst-png_s480x480&_nc_cat=104&ccb=1-7&_nc_sid=aee45a&_nc_ohc=OEfE7TGvRbcAX9Lwnqo&_nc_ad=z-m&_nc_cid=0&_nc_ht=scontent.xx&oh=03_AVJaBQXkeTQc3PbfYqHvYtXJEhUZnHG_3LUF_EvomUPjPQ&oe=62D10C9F)


![](https://scontent.xx.fbcdn.net/v/t1.15752-9/284937695_8397444896947802_7098575315767581280_n.png?stp=dst-png_p206x206&_nc_cat=109&ccb=1-7&_nc_sid=aee45a&_nc_ohc=h_6gXJaQYfsAX87Bvx-&_nc_ad=z-m&_nc_cid=0&_nc_ht=scontent.xx&oh=03_AVKzKVJLo4b6r2DPyj6_3K_JmVtxA9HT29wY-DC3cdIulw&oe=62D35345)

![](https://scontent.xx.fbcdn.net/v/t1.15752-9/285364149_1209590779795497_2438689606973803339_n.png?stp=dst-png_p206x206&_nc_cat=105&ccb=1-7&_nc_sid=aee45a&_nc_ohc=FEFsuA50wucAX_aWkCe&_nc_ad=z-m&_nc_cid=0&_nc_ht=scontent.xx&oh=03_AVKyLOeKgkxINCyuvAc6lFEvPwXVLGFWX6CUa5Cp0N2uNA&oe=62D361AE)


# Task
Prepare a payload (e.g badfile) which will cause the program to call the the *secret* function and then open a shell with root's privilege when executed by other users

# Step by Step Procedure
## Step 1: Writing an assembly code
- The return value of a function is stored in ```eax```
