# web3dvlpr
A code about smart contract for a decentralized access control system
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract AdvancedSecuritySystem {
    address public owner;

    enum Role {Admin, Moderator, User}

    mapping(address => Role) public userRoles;
    mapping(Role => mapping(string => bool)) public rolePermissions;

    event AccessGranted(address indexed user, Role role);
    event AccessRevoked(address indexed user);
    event PermissionUpdated(Role role, string permission, bool value);

    modifier onlyOwner() {
        require(msg.sender == owner, "Not the contract owner");
        _;
    }

    modifier hasRole(Role _role) {
        require(userRoles[msg.sender] == _role, "Insufficient role");
        _;
    }

    constructor() {
        owner = msg.sender;

        // Assign the owner the Admin role with all permissions
        userRoles[owner] = Role.Admin;
        rolePermissions[Role.Admin]["ManageRoles"] = true;
        rolePermissions[Role.Admin]["ManagePermissions"] = true;
        emit AccessGranted(owner, Role.Admin);
    }

    function grantAccess(address _user, Role _role) external onlyOwner {
        require(userRoles[_user] == Role.User, "User already has a role");

        userRoles[_user] = _role;
        emit AccessGranted(_user, _role);
    }

    function revokeAccess(address _user) external onlyOwner {
        require(userRoles[_user] != Role.User, "User doesn't have a role");

        delete userRoles[_user];
        emit AccessRevoked(_user);
    }

    function updatePermission(Role _role, string memory _permission, bool _value) external onlyOwner {
        rolePermissions[_role][_permission] = _value;
        emit PermissionUpdated(_role, _permission, _value);
    }

    function checkPermission(string memory _permission) external view returns (bool) {
        return rolePermissions[userRoles[msg.sender]][_permission];
    }

    function performSecureOperation() external hasRole(Role.Admin) {
        // This function can only be called by users with the Admin role
        // Perform secure operation logic here
    }

    function destroy() external onlyOwner {
        selfdestruct(payable(owner));
    }
}
