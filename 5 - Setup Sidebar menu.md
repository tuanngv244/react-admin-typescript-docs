# Setup Sidebar menu

## 1. Setup tài nguyên ảnh:

- Trong folder `assets`:
  - Tải tài nguyên ảnh theme tại [Theme](https://vitejs.dev/) bỏ vảo folder `assets`.
  - Tạo file `assets/index.ts` export ảnh.

```jsx
// assets/index.ts
export { ReactComponent as LogoLight } from "./ic-light-logo.svg";
export { ReactComponent as LogoDark } from "./ic-dark-logo.svg";
export { ReactComponent as LogoShort } from "./ic-short-logo.svg";
export { ReactComponent as IconAccount } from "./ic-icon-account.svg";
```

## 2. Setup components:

- Chạy lệnh `yarn add @tabler/icons-react simplebar-react @mui/icons-material` để cài đặt thư viện icon và scrollbar.
- Trong folder `components`:
  - Tạo component `Logo`.
    - Dùng `customizer.activeMode` kiểm tra điều kiện render logo light/dark mode.
  - Tạo component `Scrollbar`:
    - Dùng để custom scrollbar cho dự án.
  - Tạo component `Header`:
    - Trong component `Header` tạo component `ProfileHeader.tsx`.

```jsx
// Logo/index.tsx
import { FC } from "react";
import { Link } from "react-router-dom";
import { AppState } from "@/store";
import { styled } from "@mui/material";
import { useSelector } from "react-redux";
import { SVGS } from "@/assets/icons";

const Logo: FC = () => {
  const customizer = useSelector((state: AppState) => state.customizer);

  const LinkStyled = styled(Link)(() => ({
    height: customizer.TopbarHeight,
    width: customizer.isCollapse ? "40px" : "180px",
    overflow: "hidden",
    display: "block",
  }));

  if (customizer?.isCollapse) {
    return (
      <LinkStyled
        to="/"
        style={{
          display: "flex",
          alignItems: "center",
        }}
      >
        <SVGS.ICLogoShort />
      </LinkStyled>
    );
  }

  return (
    <LinkStyled
      to="/"
      style={{
        display: "flex",
        alignItems: "center",
      }}
    >
      {customizer.activeMode === "dark" ? (
        <SVGS.ICLogoLight />
      ) : (
        <SVGS.ICLogoDark />
      )}
    </LinkStyled>
  );
};

export default Logo;
```

```jsx
// Scrollbar/index.tsx
import SimpleBar from "simplebar-react";
import "simplebar-react/dist/simplebar.min.css";

import { Box, styled, SxProps } from "@mui/material";

const SimpleBarStyle = styled(SimpleBar)(() => ({
  maxHeight: "100%",
}));

interface PropsType {
  children: React.ReactElement | React.ReactNode;
  sx: SxProps;
}

const Scrollbar = (props: PropsType) => {
  const { children, sx, ...other } = props;
  const isMobile =
    /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
      navigator.userAgent
    );

  if (isMobile) {
    return <Box sx={{ overflowX: "auto" }}>{children}</Box>;
  }

  return (
    <SimpleBarStyle sx={sx} {...other}>
      {children}
    </SimpleBarStyle>
  );
};

export default Scrollbar;
```

```jsx
// Header/ProfileHeader.tsx
import {
  Avatar,
  Box,
  Button,
  Divider,
  IconButton,
  Menu,
  Stack,
  Typography,
} from "@mui/material";
import { useState } from "react";
import { Link } from "react-router-dom";
import { IconMail } from "@tabler/icons-react";
import ProfileImg from "@/assets/profile/user-1.jpg";
import { useTranslation } from "react-i18next";

const ProfileHeader = () => {
  const [modalSetting, setModalSetting] = useState(null);
  const { t } = useTranslation();
  const showModal = (event: any) => {
    setModalSetting(event.currentTarget);
  };
  const handleCloseModal = () => {
    setModalSetting(null);
  };

  const _onLogout = () => {
    // Code function logout here
  };

  return (
    <Box>
      <IconButton
        size="large"
        aria-label="show 11 new notifications"
        color="inherit"
        aria-controls="msgs-menu"
        aria-haspopup="true"
        sx={{
          ...(typeof modalSetting === "object" && {
            color: "primary.main",
          }),
        }}
        onClick={showModal}
      >
        <Avatar
          src={ProfileImg}
          alt={ProfileImg}
          sx={{
            width: 35,
            height: 35,
          }}
        />
      </IconButton>
      <Menu
        id="msgs-menu"
        anchorEl={modalSetting}
        keepMounted
        open={Boolean(modalSetting)}
        onClose={handleCloseModal}
        anchorOrigin={{ horizontal: "right", vertical: "bottom" }}
        transformOrigin={{ horizontal: "right", vertical: "top" }}
        sx={{
          "& .MuiMenu-paper": {
            width: "360px",
            p: 4,
          },
        }}
      >
        <Typography variant="h5">User Profile</Typography>
        <Stack direction="row" py={3} spacing={2} alignItems="center">
          <Avatar
            src={ProfileImg}
            alt={ProfileImg}
            sx={{ width: 95, height: 95 }}
          />
          <Box>
            <Typography
              variant="subtitle2"
              color="textPrimary"
              fontWeight={600}
            >
              Tran Nghia
            </Typography>
            <Typography variant="subtitle2" color="textSecondary">
              CEO & Founder
            </Typography>
            <Typography
              variant="subtitle2"
              color="textSecondary"
              display="flex"
              alignItems="center"
              gap={1}
            >
              <IconMail width={15} height={15} />
              contact@cfdstudio.vn
            </Typography>
          </Box>
        </Stack>
        <Divider />
        <Box key={"Profile"}>
          <Box sx={{ py: 2, px: 0 }} className="hover-text-primary">
            <Link to={"/"}>
              <Stack direction="row" spacing={2}>
                <Box
                  width="45px"
                  height="45px"
                  bgcolor="primary.light"
                  display="flex"
                  alignItems="center"
                  justifyContent="center"
                >
                  <Avatar
                    alt={""}
                    sx={{
                      width: 24,
                      height: 24,
                      borderRadius: 0,
                    }}
                  />
                </Box>
                <Box>
                  <Typography
                    variant="subtitle2"
                    fontWeight={600}
                    color="textPrimary"
                    className="text-hover"
                    noWrap
                    sx={{
                      width: "240px",
                    }}
                  >
                    {t("My Profile")}
                  </Typography>
                  <Typography
                    color="textSecondary"
                    variant="subtitle2"
                    sx={{
                      width: "240px",
                    }}
                    noWrap
                  >
                    {t("Account Settings")}
                  </Typography>
                </Box>
              </Stack>
            </Link>
          </Box>
        </Box>
        <Box mt={2}>
          <Button
            variant="outlined"
            color="primary"
            fullWidth
            onClick={_onLogout}
          >
            Logout
          </Button>
        </Box>
      </Menu>
    </Box>
  );
};

export default ProfileHeader;
```

- Trong `Header`:
  - Dùng `toggleSidebar` và `toggleMobileSidebar` để bật tắt sidebar menu.

```jsx
// Header/index.tsx
import {
  AppBar,
  Box,
  IconButton,
  Stack,
  Toolbar,
  styled,
  useMediaQuery,
} from "@mui/material";
import { IconMenu2 } from "@tabler/icons-react";
import { AppState } from "@/store";
import { useDispatch, useSelector } from "react-redux";
import {
  toggleMobileSidebar,
  toggleSidebar,
} from "@/store/customizer/CustomizerSlice";
import ProfileHeader from "@/components/Header/ProfileHeader";

const Header = () => {
  // Check responsive
  const lgUp = useMediaQuery((theme: any) => theme.breakpoints.up("lg"));
  // drawer
  const customizer = useSelector((state: AppState) => state.customizer);
  const dispatch = useDispatch();

  const AppBarStyled = styled(AppBar)(({ theme }) => ({
    boxShadow: "none",
    background: theme.palette.background.paper,
    justifyContent: "center",
    backdropFilter: "blur(4px)",
    border: "none",
    [theme.breakpoints.up("lg")]: {
      minHeight: customizer.TopbarHeight,
    },
  }));

  const ToolbarStyled = styled(Toolbar)(({ theme }) => ({
    width: "100%",
    color: theme.palette.text.secondary,
  }));

  return (
    <AppBarStyled position="sticky" color="default">
      <ToolbarStyled>
        {/* ------------------------------------------- */}
        {/* Toggle Button Sidebar */}
        {/* ------------------------------------------- */}
        <IconButton
          color="inherit"
          aria-label="menu"
          onClick={
            lgUp
              ? () => dispatch(toggleSidebar())
              : () => dispatch(toggleMobileSidebar())
          }
        >
          <IconMenu2 size="20" />
        </IconButton>
        <Box flexGrow={1} />
        <Stack spacing={1} direction="row" alignItems="center">
          <ProfileHeader />
        </Stack>
      </ToolbarStyled>
    </AppBarStyled>
  );
};

export default Header;
```

- Tạo component `Customizer`.

```jsx
import Scrollbar from "@/components/Scrollbar";
import { AppState } from "@/store";
import { setDarkMode, setTheme } from "@/store/customizer/CustomizerSlice";
import DarkModeTwoToneIcon from "@mui/icons-material/DarkModeTwoTone";
import WbSunnyTwoToneIcon from "@mui/icons-material/WbSunnyTwoTone";
import { ThemeColor } from "@/constants/theme";
import {
  Divider,
  Drawer,
  Fab,
  Grid,
  IconButton,
  Stack,
  Tooltip,
  Typography,
  styled,
} from "@mui/material";
import Box, { BoxProps } from "@mui/material/Box";
import { IconCheck, IconSettings, IconX } from "@tabler/icons-react";
import { FC, useState } from "react";
import { useDispatch, useSelector } from "react-redux";

const SidebarWidth = "320px";
interface colors {
  id: number;
  bgColor: string;
  disp?: string;
}
const Customizer: FC = () => {
  const [showDrawer, setShowDrawer] = useState(false);
  const customizer = useSelector((state: AppState) => state.customizer);

  const dispatch = useDispatch();

  const StyledBox =
    styled(Box) <
    BoxProps >
    (({ theme }) => ({
      boxShadow: theme.shadows[8],
      padding: "20px",
      cursor: "pointer",
      justifyContent: "center",
      display: "flex",
      transition: "0.1s ease-in",
      border: "1px solid rgba(145, 158, 171, 0.12)",
      "&:hover": {
        transform: "scale(1.05)",
      },
    }));

  const thColors: colors[] = [
    {
      id: 1,
      bgColor: "#5D87FF",
      disp: ThemeColor.BLUE_THEME,
    },
    {
      id: 2,
      bgColor: "#0074BA",
      disp: ThemeColor.AQUA_THEME,
    },
    {
      id: 3,
      bgColor: "#763EBD",
      disp: ThemeColor.PURPLE_THEME,
    },
    {
      id: 4,
      bgColor: "#0A7EA4",
      disp: ThemeColor.GREEN_THEME,
    },
    {
      id: 5,
      bgColor: "#01C0C8",
      disp: ThemeColor.CYAN_THEME,
    },
    {
      id: 6,
      bgColor: "#FA896B",
      disp: ThemeColor.ORANGE_THEME,
    },
  ];

  return (
    <div>
      {/* Button to open customizer */}
      <Tooltip title="Settings">
        <Fab
          color="primary"
          aria-label="settings"
          sx={{ position: "fixed", right: "25px", bottom: "15px" }}
          onClick={() => setShowDrawer(true)}
        >
          <IconSettings stroke={1.5} />
        </Fab>
      </Tooltip>
      <Drawer
        anchor="right"
        open={showDrawer}
        onClose={() => setShowDrawer(false)}
        PaperProps={{
          sx: {
            width: SidebarWidth,
          },
        }}
      >
        {/* Customizer Sidebar */}
        <Scrollbar sx={{ height: "calc(100vh - 5px)" }}>
          <Box
            p={2}
            display="flex"
            justifyContent={"space-between"}
            alignItems="center"
          >
            <Typography variant="h4">Settings</Typography>

            <IconButton color="inherit" onClick={() => setShowDrawer(false)}>
              <IconX size="1rem" />
            </IconButton>
          </Box>
          <Divider />
          <Box p={3}>
            {/* Dark light theme setting */}
            <Typography variant="h6" gutterBottom>
              Theme Option
            </Typography>
            <Stack direction={"row"} gap={2} my={2}>
              <StyledBox
                onClick={() => dispatch(setDarkMode("light"))}
                display="flex"
                gap={1}
              >
                <WbSunnyTwoToneIcon
                  color={
                    customizer.activeMode === "light" ? "primary" : "inherit"
                  }
                />
                Light
              </StyledBox>
              <StyledBox
                onClick={() => dispatch(setDarkMode("dark"))}
                display="flex"
                gap={1}
              >
                <DarkModeTwoToneIcon
                  color={
                    customizer.activeMode === "dark" ? "primary" : "inherit"
                  }
                />
                Dark
              </StyledBox>
            </Stack>
            <Box pt={3} />

            {/* Theme Color setting */}
            <Typography variant="h6" gutterBottom>
              Theme Colors
            </Typography>
            <Grid container spacing={2}>
              {thColors.map((thcolor) => (
                <Grid item xs={4} key={thcolor.id}>
                  <StyledBox onClick={() => dispatch(setTheme(thcolor.disp))}>
                    <Tooltip title={`${thcolor.disp}`} placement="top">
                      <Box
                        sx={{
                          backgroundColor: thcolor.bgColor,
                          width: "25px",
                          height: "25px",
                          borderRadius: "60px",
                          alignItems: "center",
                          justifyContent: "center",
                          display: "flex",
                          color: "white",
                        }}
                        aria-label={`${thcolor.bgColor}`}
                      >
                        {customizer.activeTheme === thcolor.disp ? (
                          <IconCheck width={13} />
                        ) : (
                          ""
                        )}
                      </Box>
                    </Tooltip>
                  </StyledBox>
                </Grid>
              ))}
            </Grid>
          </Box>
        </Scrollbar>
      </Drawer>
    </div>
  );
};

export default Customizer;
```

## 3. Setup Sibar menu:

- Trong folder `components`:
  - Tạo component `Sidebar`.

```jsx
// components/Sidebar/index.tsx
const Sidebar = () => {
  return <div>...</div>;
};
export default Sidebar;
```

- Trong component `Sidebar`:
  - Tạo component `NavCollapse`, `NavGroup`, `NavItem` để render các loại menu item khác nhau.
  - Tạo component `ProfileMenuSidebar` show thông tin user.
  - Tạo component `SidebarMain` dùng để render list menu items.

```jsx
// Sidebar/NavItem/index.tsx
import React from "react";
import { NavLink } from "react-router-dom";

import {
  ListItemIcon,
  List,
  styled,
  ListItemText,
  Chip,
  useTheme,
  Typography,
  ListItemButton,
} from "@mui/material";

import { useTranslation } from "react-i18next";
import { useSelector } from "react-redux";
import { AppState } from "@/store";

type NavGroup = {
  [x: string]: any,
  id?: string,
  navlabel?: boolean,
  subheader?: string,
  title?: string,
  icon?: any,
  href?: string,
  children?: NavGroup[],
  chip?: string,
  chipColor?: any,
  variant?: string | any,
  external?: boolean,
  level?: number,
  onClick?: React.MouseEvent<HTMLButtonElement, MouseEvent>,
};

interface ItemType {
  item: NavGroup;
  hideMenu?: any;
  onClick: (event: React.MouseEvent<HTMLElement>) => void;
  level?: number | any;
  pathDirect: string;
}

const NavItem = ({ item, level, pathDirect, hideMenu, onClick }: ItemType) => {
  const Icon = item?.icon;
  const theme = useTheme();
  const { t } = useTranslation();
  const customizer = useSelector((state: AppState) => state.customizer);

  const itemIcon =
    level > 1 ? (
      <Icon stroke={1.5} size="1rem" />
    ) : (
      <Icon stroke={1.5} size="1.3rem" />
    );

  // Re style List
  const ListItemStyled = styled(ListItemButton)(() => ({
    whiteSpace: "nowrap",
    marginBottom: "2px",
    padding: "8px 10px",
    borderRadius: `${customizer.borderRadius}px`,
    backgroundColor: level > 1 ? "transparent !important" : "inherit",
    color:
      level > 1 && pathDirect === item?.href
        ? `${theme.palette.primary.main}!important`
        : theme.palette.text.secondary,
    paddingLeft: hideMenu ? "10px" : level > 2 ? `${level * 15}px` : "10px",
    "&:hover": {
      backgroundColor: theme.palette.primary.light,
      color: theme.palette.primary.main,
    },
    "&.Mui-selected": {
      color: "white",
      backgroundColor: theme.palette.primary.main,
      "&:hover": {
        backgroundColor: theme.palette.primary.main,
        color: "white",
      },
    },
  }));

  const listItemProps: {
    component: any,
    href?: string,
    target?: any,
    to?: any,
  } = {
    component: item?.external ? "a" : NavLink,
    to: item?.href,
    href: item?.external ? item?.href : "",
    target: item?.external ? "_blank" : "",
  };

  return (
    <List component="li" disablePadding key={item?.id && item.title}>
      <ListItemStyled
        {...listItemProps}
        disabled={item?.disabled}
        selected={pathDirect === item?.href}
        onClick={onClick}
      >
        <ListItemIcon
          sx={{
            minWidth: "36px",
            p: "3px 0",
            color:
              level > 1 && pathDirect === item?.href
                ? `${theme.palette.primary.main}!important`
                : "inherit",
          }}
        >
          {itemIcon}
        </ListItemIcon>
        <ListItemText>
          {hideMenu ? "" : <>{t(`${item?.title}`)}</>}
          <br />
          {item?.subtitle ? (
            <Typography variant="caption">
              {hideMenu ? "" : item?.subtitle}
            </Typography>
          ) : (
            ""
          )}
        </ListItemText>

        {!item?.chip || hideMenu ? null : (
          <Chip
            color={item?.chipColor}
            variant={item?.variant ? item?.variant : "filled"}
            size="small"
            label={item?.chip}
          />
        )}
      </ListItemStyled>
    </List>
  );
};

export default NavItem;
```

```jsx
// Sidebar/NavGroup/index.tsx
import { ListSubheader, styled, Theme } from "@mui/material";
import { IconDots } from "@tabler/icons-react";

type NavGroup = {
  navlabel?: boolean,
  subheader?: string,
};

interface ItemType {
  item: NavGroup;
  hideMenu: string | boolean;
}

const NavGroup = ({ item, hideMenu }: ItemType) => {
  const ListSubheaderStyle = styled((props: Theme | any) => (
    <ListSubheader disableSticky {...props} />
  ))(({ theme }) => ({
    ...theme.typography.overline,
    fontWeight: "700",
    marginTop: theme.spacing(3),
    marginBottom: theme.spacing(0),
    color: "text.Primary",
    lineHeight: "26px",
    padding: "3px 12px",
    marginLeft: hideMenu ? "" : "-10px",
  }));

  return (
    <ListSubheaderStyle>
      {hideMenu ? <IconDots size="14" /> : item?.subheader}
    </ListSubheaderStyle>
  );
};

export default NavGroup;
```

```jsx
// Sidebar/NavCollapse/index.tsx
import React, { useEffect, useState } from "react";
import { useLocation } from "react-router-dom";

import {
  Collapse,
  ListItemButton,
  ListItemIcon,
  ListItemText,
  styled,
  useTheme,
} from "@mui/material";

import { AppState } from "@/store";
import { IconChevronDown, IconChevronUp } from "@tabler/icons-react";
import { useTranslation } from "react-i18next";
import { useSelector } from "react-redux";
import NavItem from "../NavItem";

type NavGroupProps = {
  [x: string]: any;
  navlabel?: boolean;
  subheader?: string;
  title?: string;
  icon: any;
  href?: string;
};

interface NavCollapseProps {
  menu: NavGroupProps;
  level: number;
  pathWithoutLastPart: any;
  pathDirect: any;
  hideMenu: any;
  onClick: (event: React.MouseEvent<HTMLElement>) => void;
}

const NavCollapse = ({
  menu,
  level,
  pathWithoutLastPart,
  pathDirect,
  hideMenu,
  onClick,
}: NavCollapseProps) => {
  const Icon = menu.icon;
  const customizer = useSelector((state: AppState) => state.customizer);
  const theme = useTheme();
  const { pathname } = useLocation();
  const { t } = useTranslation();
  const [open, setOpen] = useState(true);
  const menuIcon =
    level > 1 ? (
      <Icon stroke={1.5} size="1rem" />
    ) : (
      <Icon stroke={1.5} size="1.3rem" />
    );

  const handleClick = () => {
    setOpen(!open);
  };

  const ListItemStyled = styled(ListItemButton)(() => ({
    marginBottom: "2px",
    padding: "8px 10px",
    paddingLeft: hideMenu ? "10px" : level > 2 ? `${level * 15}px` : "10px",
    backgroundColor: open && level < 2 ? theme.palette.primary.main : "",
    whiteSpace: "nowrap",
    "&:hover": {
      backgroundColor:
        pathname.includes(menu.href as string) || open
          ? theme.palette.primary.main
          : theme.palette.primary.light,
      color:
        pathname.includes(menu.href as string) || open
          ? "white"
          : theme.palette.primary.main,
    },
    color:
      open && level < 2
        ? "white"
        : `inherit` && level > 1 && open
        ? theme.palette.primary.main
        : theme.palette.text.secondary,
    borderRadius: `${customizer.borderRadius}px`,
  }));

  // menu collapse for sub-levels
  useEffect(() => {
    setOpen(false);
    menu?.children?.forEach((item: any) => {
      if (item?.href === pathname) {
        setOpen(true);
      }
    });
  }, [pathname, menu.children]);

  // If Menu has Children
  const submenus = menu.children?.map((item: any) => {
    if (item.children) {
      return (
        <NavCollapse
          key={item?.id}
          menu={item}
          level={level + 1}
          pathWithoutLastPart={pathWithoutLastPart}
          pathDirect={pathDirect}
          hideMenu={hideMenu}
          onClick={onClick}
        />
      );
    }
    return (
      <NavItem
        key={item.id}
        item={item}
        level={level + 1}
        pathDirect={pathDirect}
        hideMenu={hideMenu}
        onClick={onClick}
      />
    );
  });

  return (
    <>
      <ListItemStyled
        onClick={handleClick}
        selected={pathWithoutLastPart === menu.href}
        key={menu?.id}
      >
        <ListItemIcon
          sx={{
            minWidth: "36px",
            p: "3px 0",
            color: "inherit",
          }}
        >
          {menuIcon}
        </ListItemIcon>
        <ListItemText color="inherit">
          {hideMenu ? "" : <>{t(`${menu.title}`)}</>}
        </ListItemText>
        {!open ? (
          <IconChevronDown size="1rem" />
        ) : (
          <IconChevronUp size="1rem" />
        )}
      </ListItemStyled>
      <Collapse in={open} timeout="auto" unmountOnExit>
        {submenus}
      </Collapse>
    </>
  );
};

export default NavCollapse;
```

- comonent `ProfileMenuSideBar` dùng show thông tin user và function `_onLogout`.

```jsx
// Sidebar/ProfileMenuSideBar/index.tsx
import {
  Avatar,
  Box,
  IconButton,
  Tooltip,
  Typography,
  useMediaQuery,
} from "@mui/material";

import img1 from "@/assets/profile/user-1.jpg";
import { AppState } from "@/store";
import { IconPower } from "@tabler/icons-react";
import { useSelector } from "react-redux";

export const ProfileMenuSideBar = () => {
  const customizer = useSelector((state: AppState) => state.customizer);

  const lgUp = useMediaQuery((theme: any) => theme.breakpoints.up("lg"));
  const hideMenu = lgUp
    ? customizer.isCollapse && !customizer.isSidebarHover
    : "";

  const _onLogout = () => {
    // Code function log here
  };

  return (
    <Box
      display={"flex"}
      alignItems="center"
      gap={2}
      sx={{ m: 3, p: 2, bgcolor: `${"secondary.light"}`, maxWidth: "270px" }}
    >
      {!hideMenu ? (
        <>
          <Avatar alt="Remy Sharp" src={img1} />

          <Box>
            <Typography variant="h6">Tran Nghia </Typography>
            <Typography variant="caption">CEO & Founder</Typography>
          </Box>
          <Box sx={{ ml: "auto" }}>
            <Tooltip title="Logout" placement="top">
              <IconButton
                onClick={_onLogout}
                color="primary"
                aria-label="logout"
                size="small"
              >
                <IconPower size="20" />
              </IconButton>
            </Tooltip>
          </Box>
        </>
      ) : (
        ""
      )}
    </Box>
  );
};
```

- Component `SidebarMain` dùng để render menu list item.
- Kiểm tra render theo các trường hợp sub header, sub menu và sub no menu.

```jsx
import React from "react";
import { useLocation } from "react-router-dom";
import { Box, List, useMediaQuery } from "@mui/material";

import { useDispatch, useSelector } from "react-redux";

import { toggleMobileSidebar } from "@/store/customizer/CustomizerSlice";
import { AppState } from "@/store";
import MenuItems from "./menuItems";
import NavGroup from "./NavGroup";
import NavCollapse from "./NavCollapse";
import NavItem from "./NavItem";

const SidebarMain = () => {
  const { pathname } = useLocation();
  const pathDirect = pathname;
  const pathWithoutLastPart = pathname.slice(0, pathname.lastIndexOf("/"));

  const customizer = useSelector((state: AppState) => state.customizer);

  const lgUp = useMediaQuery((theme: any) => theme.breakpoints.up("lg"));

  const hideMenu: any = lgUp
    ? customizer.isCollapse && !customizer.isSidebarHover
    : "";
  const dispatch = useDispatch();

  return (
    <Box sx={{ px: 3 }}>
      <List sx={{ pt: 0 }} className="sidebarNav">
        {MenuItems.map((item) => {
          // SubHeader
          if (item.subheader) {
            return (
              <NavGroup item={item} hideMenu={hideMenu} key={item.subheader} />
            );
            // If Sub Menu
          } else if (item.children) {
            return (
              <NavCollapse
                menu={item}
                pathDirect={pathDirect}
                hideMenu={hideMenu}
                pathWithoutLastPart={pathWithoutLastPart}
                level={1}
                key={item.id}
                onClick={() => dispatch(toggleMobileSidebar())}
              />
            );
            // Sub No Menu
          } else {
            return (
              <NavItem
                item={item}
                key={item.id}
                pathDirect={pathDirect}
                hideMenu={hideMenu}
                onClick={() => dispatch(toggleMobileSidebar())}
              />
            );
          }
        })}
      </List>
    </Box>
  );
};
export default SidebarMain;
```

- Trong component `Sidebar`:
  - Kiểm tra render sidebar theo 2 trường hợp desktop và mobile bằng `lgUp`.
  - Import `SidebarMain` vào component `Sidebar`.
  - Bật/tắt `Sidebar` bằng `toggleWidth`.

```jsx
// Sidebar/index.tsx
import { useMediaQuery, Box, Drawer, useTheme } from "@mui/material";
import Scrollbar from "@/components/Scrollbar";
import { useDispatch, useSelector } from "react-redux";
import {
  hoverSidebar,
  toggleMobileSidebar,
} from "@/store/customizer/CustomizerSlice";
import { AppState } from "@/store";
import SidebarMain from "@/components/SideBar/SidebarMain";
import Logo from "@/components/Logo";
import { ProfileMenuSideBar } from "@/components/SideBar/ProfileMenuSideBar.tsx";

const Sidebar = () => {
  // Check Device  > 1200 : Desktop
  const lgUp = useMediaQuery((theme: any) => theme.breakpoints.up("lg"));

  const customizer = useSelector((state: AppState) => state.customizer);

  const dispatch = useDispatch();

  const theme = useTheme();

  // width sidebar open and side close
  const toggleWidth =
    customizer.isCollapse && !customizer.isSidebarHover
      ? customizer.MiniSidebarWidth
      : customizer.SidebarWidth;

  const onHoverEnter = () => {
    if (customizer.isCollapse) {
      dispatch(hoverSidebar(true));
    }
  };

  const onHoverLeave = () => {
    dispatch(hoverSidebar(false));
  };

  // Case Desktop
  if (lgUp) {
    return (
      <Box
        sx={{
          width: toggleWidth,
          flexShrink: 0,
          ...(customizer.isCollapse && {
            position: "absolute",
          }),
        }}
      >
        <Drawer
          anchor="left"
          open
          onMouseEnter={onHoverEnter}
          onMouseLeave={onHoverLeave}
          variant="permanent"
          PaperProps={{
            sx: {
              transition: theme.transitions.create("width", {
                duration: theme.transitions.duration.shortest,
              }),
              width: toggleWidth,
              boxSizing: "border-box",
            },
          }}
        >
          <Box
            sx={{
              height: "100%",
            }}
          >
            <Box px={3}>
              <Logo />
            </Box>

            <Scrollbar sx={{ height: "calc(100% - 190px)" }}>
              <SidebarMain />
            </Scrollbar>
            <ProfileMenuSideBar />
          </Box>
        </Drawer>
      </Box>
    );
  }

  // Case mobile
  return (
    <Drawer
      anchor="left"
      open={customizer.isMobileSidebar}
      onClose={() => dispatch(toggleMobileSidebar())}
      variant="temporary"
      PaperProps={{
        sx: {
          width: customizer.SidebarWidth,
          border: "0 !important",
          boxShadow: (theme) => theme.shadows[8],
        },
      }}
    >
      <Box px={2}>
        <Logo />
      </Box>
      <SidebarMain />
    </Drawer>
  );
};

export default Sidebar;
```

## 4. Update style MenuLayout:

- Trong `MenuLayout` dựa vào `customizer` render theo theme.
- Import`Sidebar` vào và render sidebar menu.
- Import `Customizer` vào để bật/tắt mở ra cấu hình theme.

```jsx
import { FC } from 'react';
import { styled, Container, Box, useTheme } from '@mui/material';
import { Outlet } from 'react-router-dom';
import { useSelector } from 'react-redux';
import { AppState } from '@/store';
import Header from '@/components/Header';
import Sidebar from '@/components/SideBar';
import Customizer from '@/components/Customizer';

const MainWrapper = styled('div')(() => ({
    display: 'flex',
    minHeight: '100vh',
    width: '100%',
}));

const PageWrapper = styled('div')(() => ({
    display: 'flex',
    flexGrow: 1,
    paddingBottom: '60px',
    flexDirection: 'column',
    zIndex: 1,
    width: '100%',
    backgroundColor: 'transparent',
}));

const ContainerStyled = styled(Container)({
    maxWidth: 'none !important',
});

const MenuLayout: FC = () => {
    const customizer = useSelector((state: AppState) => state.customizer);
    const theme = useTheme();

const MenuLayout: React.FC = () => {
    return (
        <div className="MenuLayout">
            <Outlet />
        </div>
        <MainWrapper>
            <Sidebar />
            <PageWrapper
                className="page-wrapper"
                sx={{
                    ...(customizer.isCollapse && {
                        [theme.breakpoints.up('lg')]: {
                            ml: `${customizer.MiniSidebarWidth}px`,
                        },
                    }),
                }}
            >
                <Header />
                <ContainerStyled>
                    <Box sx={{ minHeight: 'calc(100vh - 170px)' }}>
                        <Outlet />
                    </Box>
                </ContainerStyled>
                <Customizer />
            </PageWrapper>
        </MainWrapper>
    );
};
```
