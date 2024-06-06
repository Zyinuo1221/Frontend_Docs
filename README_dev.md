# Home Component Documentation

## Overview

The `Home` component is designed for internal use at HireBeat to display specific UI elements conditionally based on the user's login credentials. It utilizes React, Material-UI, and AWS Amplify to manage access and render components dynamically based on authentication status.

## Table of Contents

- [Component Description](#component-description)
- [Dependencies](#dependencies)
- [Usage](#usage)
- [Code Structure](#code-structure)
- [Security and Privacy](#security-and-privacy)

## Component Description

The `Home` component is specifically designed to render the `GraphDev` component within a responsive grid layout if the authenticated user's `loginId` ends with `@hirebeat.co`. This component is intended for development purposes and is typically used for internal testing and debugging.

## Dependencies

- React
- Material-UI (MUI)
- AWS Amplify

## Usage

The component should be used within a **development environment** only and is not intended for production use. It should be placed under the `dev` folder to separate development-specific components from production components, ensuring a clean and manageable codebase.

## Code Structure

### Imports

- **Material-UI Grid:** Utilized for creating responsive grid layouts, enhancing the structure and design of the user interface.
- **GraphDev:** A custom component that likely serves as a graphical or analytical tool, specifically designed for development testing or internal data visualization.
- **AWS Amplify Auth:** Employed to fetch the current user details, which are crucial for managing access control based on user credentials.

### React Hooks

- **useState:** Manages the `allowed` state, which determines the visibility of the `GraphDev` component.
- **useEffect:** Executes side effects to fetch user data on component mount. It checks the domain of the user’s `loginId` and updates the `allowed` state accordingly.

### Conditional Rendering

The component renders content based on the `allowed` state, which is determined by checking if the user's email domain is `@hirebeat.co`. Below is the JSX code for this logic:

```jsx
return (
  allowed && (
    <Grid container spacing={6}>
      <GraphDev />
    </Grid>
  )
);
```

## Security and Privacy

This section outlines the security measures and privacy practices implemented in the `Home` component to ensure data protection and access control.

### User Authentication Check

- **Purpose:** Ensures that only users whose `loginId` ends with `@hirebeat.co` can access specific development tools within the application.
- **Implementation:** This is achieved by checking the user's email domain during the authentication process. If the domain matches `@hirebeat.co`, the user is granted access to certain features otherwise restricted.

